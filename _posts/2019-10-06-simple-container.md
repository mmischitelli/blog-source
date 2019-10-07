---
layout: post
current: post
cover:  assets/images/posts/simple-container.jpg
navigation: True
title: Simple C++ Container
date: 2019-10-06 10:00:00
tags: [cpp]
class: post-template
subclass: 'post tag-cpp'
author: mmischitelli
---

When I used to develop C# UWP application, MVVM was the architectural pattern of choice of many developers and maybe it still is.

A recurring concept in the MVVM pattern is the dependency injection, [*a 25-dollar term for a 5-cent concept*](https://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html). Dependency injection is a pretty common design pattern where classes are not allowed to create instances of external services they need to accomplish tasks. Instead, those dependencies are provided at runtime, usually in the constructor.

But I don't want to spend too much time explaining what DI is. Instead, I'm more interested in talking about the *container*, which is a fundamental component of it.

## What functionality does a DI container provide?

You can think about the container as a class whose purpose is to decouple instantiation from usage. Objects creation is no longer performed by invoking *new*; instead, it depends on some policy the developer specified. 

When does it happen? During the registration phase, usually right after the container is created. Why is this registration even needed? Because, apart from the *lifetime policy* I just mentioned, developers are also able to specify which implementation will get resolved when we ask for a particular interface.

At this point, you're probably looking at this article puzzled and confused. Let me shed some light on the subject and make things more clear.

## Ladies and gents, the Native Container

Without further ado, let's see how it all works.

```cpp
auto container = std::make_shared<NativeContainer>();
container->RegisterType<SimpleInterface, SimpleType, ContainerControlledManager>();
```

On the first line, we simply create an instance of some class named `NativeContainer`. nothing fancy. On the next line, we're invoking a templated method accepting three types:
  - `SimpleInterface`: the interface that the next argument must implement
  - `SimpleType`: some class which implements the previous interface
  - `ContainerControlledManager`: the lifetime manager I've mentioned in the previous paragraph

Before diving deeper into the inner workings of the Native Container, let me explain what is happening here.

By invoking `RegisterType`, we're telling the container that whenever we require an instance of `SimpleInterface`, it should be *resolved* by instantiating an object of `SimpleType`. How this object is created and how its memory is managed depends on which *lifetime manager* we're specifying. Note that, at this point, no instance is actually created!

Later on, to retrieve a `SimpleType` instance, we simply call:

```cpp
auto object = container->Resolve<SimpleInterface>();
```

Do you see a call to the `new` operator? Are we specifying `SimpleType` anywhere in the above line? Exactly, both questions have the same answer: nope. The Native Container is hiding the actual type of our the returned instance behind an interface and the caller has no simple way of knowing it. Moreover, as far as the caller is concerned, we're not calling `new` anywhere: this allows us to execute some custom logic at creation time; it will become more evident in the future when I'll talk about dependency resolution, but it is something for another post.

## Creating a resolvable class

As mentioned above, any type we'd like to register in the container must implement an interface that abstracts (or hides) the actual implementation. But, in turn, this interface must also derive from another interface, something specific that is tied to how the Native Container works.

I named this `IResolvableType`:

```cpp
class IResolvableType
{
public:
  virtual ~IResolvableType() = default;
  [[nodiscard]] virtual ClassUniqueId GetTypeId() const = 0;
};
```

This interface provides two core functionalities: it declares a virtual method that, when overridden by derived classes, returns some kind of *unique identifier* that can be useful at runtime to perform type checking cheaply. This is not meant to be as comprehensive as RTTI, nor is really necessary, but for the sake of completeness, it's worth mentioning.

The other handy functionality lies in the fact that all the resolvable interfaces must derive from it: we can thus perform compile-time checks to ensure that only compatible types and interfaces are being registered in the container.

Unfortunately, implementing the `IResolvableType` is not enough to guarantee what we're trying to achieve. Here's the complete declaration of a very simple interface that can be registered in the Native Container:

```cpp
class PrinterInterface : public IResolvableType
{
  static constexpr char m_kTypeId = 0;

  template<typename T>
  static constexpr bool SameAs() { 
    if constexpr (&m_kTypeId == &T::m_kTypeId) { 
      return true; 
    } 
    return false; 
  }

  static constexpr ClassUniqueId GetStaticTypeId() { 
    return static_cast<ClassUniqueId>(&m_kTypeId); 
  }

  [[nodiscard]] ClassUniqueId GetTypeId() const override { 
    return GetStaticTypeId(); 
  }

public:
  virtual void PrintSomething() = 0;
};
```

The whole runtime type comparison concept is taken straight from [the templated properties]({% post_url 2019-07-17-templated-properties %}/#generic-properties) article and evolved a bit to be `constexpr`. Since I'm not fond of duplicating code and consider the whole type-checking bits to be quite distracting, I prefer putting them in a handy macro:

```cpp
#define ResolvableMagic(ClassType) \
public: \
	static constexpr char m_kTypeId = 0; \
	template<typename T> \
	static constexpr bool SameAs() { if constexpr (&m_kTypeId == &T::m_kTypeId) { return true; } return false; } \
	static constexpr ClassUniqueId GetStaticTypeId() { return static_cast<ClassUniqueId>(&m_kTypeId); } \
	[[nodiscard]] ClassUniqueId GetTypeId() const override { return GetStaticTypeId(); } \
private:\
```

This allows me to finally declare the interface and the implementation as you would expect:
```cpp
class PrinterInterface : public IResolvableType
{
  ResolvableMagic(PrinterInterface)
public:
  virtual void PrintSomething() = 0;
}

class HelloWorldPrinter : public PrinterInterface
{
public:
  void PrintSomething() override { cout << "Hello, world!" << endl; }
};
```

Much more readable, isn't it? It also puts back the focus to what this interface provides: a method that, when called, prints something. Also quite uninspiringly, the `HelloWorldPrinter` implementation prints... an hello world :)

## The Native Container

Ok so, at this point we're able to make *resolvable* types and interfaces.. but how do we actually *resolve* them? Where do we ask for the uninspirigly yet awesome `HelloWorldPrinter` created above?

The native container has the answer, of course!

```cpp
class NativeContainer
{
	std::map<ClassUniqueId, std::shared_ptr<ILifetimeManager>> m_RegisteredInstances;

public:

	template<typename Interface, typename Type, template<typename> class LifetimeManager>
	void RegisterType()
	{
		static_assert((std::is_base_of<ILifetimeManager, LifetimeManager<Type>>::value), 
			"The LifetimeManager class must derive from ILifetimeManager");
		static_assert((std::is_base_of<IResolvableType, Interface>::value), 
			"The interface you are trying to register must derive from IResolvableType");
		static_assert((std::is_base_of<Interface, Type>::value), 
			"The type you are trying to register must derive from the interface you provided as Interface");

		auto lifetimeManager = std::make_shared<LifetimeManager<Type> >();
		m_RegisteredInstances.insert(std::make_pair(Interface::GetStaticTypeId(), lifetimeManager));
	}

	template<typename Interface>
	[[nodiscard]] std::shared_ptr<Interface> Resolve() const
	{
		static_assert((std::is_base_of<IResolvableType, Interface>::value), 
			"The interface you are trying to register must derive from IResolvableType");

		const auto kManager = m_RegisteredInstances.find(Interface::GetStaticTypeId());
		if (kManager == m_RegisteredInstances.end()) {
			return {};
		}
		return kManager->second->GetInstance<Interface>();
	}
};
```

I'm asking you, the reader, a little bit of faith at this point. The container is not that complex if you skip through the static checks. It simply declares two methods, one that is used to register types and interfaces; the other one to retrieve (or *resolve*) a type's instance.

The reason why this class is so simple is that it does not actually create instances. The `RegisterType` method just writes a pair inside a map, linking together the interface's unique identifier with an object, the *lifetime manager*.

We're not going to see how these managers are created or what they do at this time, hence the leap of faith I've asked you before: it would make this article too long. But as can be easily deduced from the above code, when it's invoked in the `Resolve` method, it just returns an instance of the type it was declared with (it's a templated class!).

## Conclusions

With a bit of templated magic, I was able to pull off this (simple and limited) container. It represents a good starting point for further research: for example, these kinds of containers are usually able to resolve dependencies a type might have to work properly.

An RSS reader might depend on some kind of download service; a class might need a logger service to support debugging. My implementation doesn’t support dependency resolution but it could be easily implemented if the type exposed a list of unique identifiers for each dependency, along with storage for the resolved instances… so that the container, before returning the type’s instance, might read those IDs, resolve and then inject them in it.

In the next few weeks, I’m planning to write a new post to talk about the lifetime managers we’ve seen above. It’ll be interesting to see how the container offloads object creation in separate classes…