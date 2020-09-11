---
layout: post
current: post
cover:  assets/images/posts/lifetime-managers.jpg
navigation: True
title: Lifetime managers
date: 2019-11-14 10:00:00
tags: [cpp]
class: post-template
subclass: 'post tag-cpp'
author: mmischitelli
---

Follow up post on a possible implementation of a [simple c++ container]({% post_url 2019-10-06-simple-container %}).

After having seen how to implement a simple yet functioning dependency injection container, we're going to see how to make use of it by introducing the concept of *lifetime managers*.

## Managing how long your objects should live

Lifetime managers serve just that purpose. Asking the resolver for two objects sharing the same kind of manager will cause those objects to be destroyed at the same time.
Or, to see it from another perspective, we're categorizing objects by the way they're destroyed.

In this post, I will show you how to implement three different kinds of managers that are often found in DI containers:
* Container-controlled
* Externally-controlled
* Per-resolve

## A common ground
Before diving into how the various managers are implemented and what they do, it's useful seeing the common ancestor they all derive from, the `ILifetimeManager` interface:

```cpp
class ILifetimeManager
{
	friend class NativeContainer;
	
	template<typename InterfaceType>
	[[nodiscard]] std::shared_ptr<InterfaceType> GetInstance() {
		return std::static_pointer_cast<InterfaceType>(_GetInstance());
	}

protected:
	[[nodiscard]] virtual std::shared_ptr<void> _GetInstance() = 0;

public:
	ILifetimeManager() = default;
	virtual ~ILifetimeManager() = default;

	ILifetimeManager(const ILifetimeManager&) 	= delete;
	ILifetimeManager(ILifetimeManager&&) 		= delete;
	void operator=(const ILifetimeManager&) 	= delete;
	void operator=(ILifetimeManager&&) 			= delete;
};
```

The first thing to notice is that its most important method, `GetInstance`, is private. On the other hand, the interface declares the `NativeContainer` to be its friend. Only the `NativeContainer` will thus be able to get an instance of the type associated with this container.

Another important aspect to highlight is that `ILifetimeManager` only really needs to know the interface it's going to resolve. It will be the templated, derived manager that's going to store the real type in a template argument.


## Container-controlled
At this point, we can finally dive into the actual managers.

The Container-controlled lifetime manager is usually associated with the Singleton pattern. Every time we're going to ask for an instance of the stored type, it will always produce the same object. The lifetime of said object, however, is not tied to the whole application, like in a true singleton. It will, instead, be tied to the lifetime of the container producing this instance.

Having multiple containers will give you the possibility of having multiple instances of the type stored in this lifetime manager.

```cpp
template<typename DerivedType>
class ContainerControlledManager final : public ILifetimeManager
{
	std::shared_ptr<DerivedType> m_Instance;

	[[nodiscard]] std::shared_ptr<void> _GetInstance() override
	{
		if (m_Instance == nullptr) {
			m_Instance = std::make_shared<DerivedType>();
		}
		return std::static_pointer_cast<void>(m_Instance);
	}
};
```

But how does it work? It's really simple actually: it keeps a shared pointer of the instance it's going to return every time `GetInstance` is called. Is that simple!

By using `static_pointer_cast` we're also making sure that the reference count of the original shared pointer is preserved. It is perfectly safe to hide the derived type behind a void pointer as it is then converted back to the interface type in the `ILifetimeManager::GetInstance` method. The shared pointer will thus know how to properly destroy the object.

```cpp
BOOST_AUTO_TEST_CASE(container_controlled_instance)
{
	auto container = std::make_shared<NativeContainer>();
	container->RegisterType<SimpleInterface, SimpleType, ContainerControlledManager>();

	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		BOOST_TEST(simpleTest->GetCounter() == 2);
	}
}
```

Running the above test case verifies that `GetCounter` does return 2.

## Externally-controlled

As the name suggests, this type of manager doesn't control the lifetime of the objects it's creating. Instead, it keeps a weak reference to it.

```cpp
template<typename DerivedType>
class ExternallyControlledManager final : public ILifetimeManager
{
	std::weak_ptr<void> m_Instance;

	[[nodiscard]] std::shared_ptr<void> _GetInstance() override
	{
		if (!m_Instance.expired()) {
			return m_Instance.lock();
		}
		std::shared_ptr<void> instance = std::shared_ptr<void>(new DerivedType());
		m_Instance = instance;
		return instance;
	}
};
```

But as we have seen, the Native Container always returns a shared pointer. This implies that as long as the caller keeps a strong reference to the returned instance, the internal weak pointer will remain valid. Whenever the strong reference is destroyed, the weak pointer will become invalid, causing the manager to produce a new instance if asked.

```cpp
BOOST_AUTO_TEST_CASE(container_externally_managed_instance_1)
{
	auto container = std::make_shared<NativeContainer>();
	container->RegisterType<SimpleInterface, SimpleType, ExternallyControlledManager>();

	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		BOOST_TEST(simpleTest->GetCounter() == 0);
	}
}
```

Running the above test case verifies that `GetCounter` does return 0, as the previous instances of SimpleInterface are destroyed when exiting the scope they've been resolved into.

```cpp
BOOST_AUTO_TEST_CASE(container_externally_managed_instance_2)
{
	auto container = std::make_shared<NativeContainer>();
	container->RegisterType<SimpleInterface, SimpleType, ExternallyControlledManager>();

	auto storage = container->Resolve<SimpleInterface>();
	storage->IncrementCounter();

	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		BOOST_TEST(simpleTest->GetCounter() == 3);
	}
}
```

This other test, however, verifies that `GetCounter` returns 3. The first instance resolved is kept during the whole test, causing all the following resolutions to always return the same object.

## Per-resolve
The simplest possible type of manager is this one. It acts just like a factory, returning a new instance every time we're asking for it. No checks, no questions asked.

```cpp
template<typename DerivedType>
class PerResolveManager final : public ILifetimeManager
{
	[[nodiscard]] std::shared_ptr<void> _GetInstance() override {
		return std::shared_ptr<void>(new DerivedType());
	}
};
```

And to demonstrate how it works...

```cpp
BOOST_AUTO_TEST_CASE(container_per_resolve_instance)
{
	auto container = std::make_shared<NativeContainer>();
	container->RegisterType<SimpleInterface, SimpleType, PerResolveManager>();

	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		simpleTest->IncrementCounter();
	}
	{
		auto simpleTest = container->Resolve<SimpleInterface>();
		BOOST_TEST(simpleTest->GetCounter() == 0);
	}
}
```

As you can see, the test is exactly like the first one we've seen for the [Externally-controlled](#externally-controlled) case. It verifies that `GetCounter` does return 0, as we're always creating new instances.

## Conclusions
I hope this container-and-managers article in two posts has been interesting to read. I surely had fun creating it an see it working. As mentioned in the [previous post]({% post_url 2019-10-06-simple-container %}), the Native Container to be really useful should support dependency resolution.

I hope someone will carry on and improve my design. For this reason, I'm making the whole project available on [GitHub](https://github.com/mmischitelli/NativeContainer). Thanks for reading, see you in the next post!