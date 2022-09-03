![Pool Diagram](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/Pool_diagram.png)

Object pooling aims to alleviate the performance hit of instantiating/destroying many objects by instantiating them beforehand, and then activating/deactivating the objects instead, reusing objects when needed.

This is a common pattern in videogames even outside of Unity, but we have enriched it by using some Unity-specific features like ScriptableObjects as templates to setup pools directly in the Editor, without code.

## Table of Contents
- [Creating a factory for the new type](#creating-a-factory-for-the-new-type)
- [Creating a pool for the new type](#creating-a-pool-for-the-new-type)
- [Using the pool](#using-the-pool)
- [More info](#more-info)

### Creating a factory for the new type

```cs
using UnityEngine;
using UOP1.Factory;

[CreateAssetMenu(fileName = "MyComponentFactory", menuName = "Factory/MyComponent")]
public class MyComponentFactorySO : FactorySO<MyComponent>
{
    public override MyComponent Create(){
        return new GameObject("MyComponent").AddComponent<MyComponent>();
    }
}
```

Also, check out the page for the [Factory implementation](https://github.com/UnityTechnologies/open-project-1/wiki/Factory) we have created for the game.

### Creating a pool for the new type

```cs
using UnityEngine;
using UOP1.Pool;
using UOP1.Factory;

[CreateAssetMenu(fileName = "MyComponentPool", menuName = "Pool/MyComponent")]
public class MyComponentPoolSO : ComponentPoolSO<MyComponent>
{
	[SerializeField]
	private MyComponentFactorySO _factory;
	[SerializeField]
	private int _initialPoolSize;

	public override IFactory<MyComponent> Factory
	{
		get
		{
			return _factory;
		}
		set
		{
			_factory = value as MyComponentFactorySO;
		}
	}

	public override int InitialPoolSize
	{
		get
		{
			return _initialPoolSize;
		}
		set
		{
			_initialPoolSize = value;
		}
	}
}
```

### Using the pool

There are two ways to use a pool. Either you can create the pool and factory as an asset in the project and set it to an object reference in the inspector or you can create the pool and factory at runtime.

As an asset:

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MyPoolManager : MonoBehaviour
{
	[SerializeField]
	private MyComponentPoolSO _pool = default;

	private void Start()
	{
		MyComponent myComponent = _pool.Request();
        //Do something with it
		_pool.Return(myComponent);
	}
}
```

At runtime:

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MyRuntimePoolManager : MonoBehaviour
{
	[SerializeField]
	private int _initialPoolSize = 5;

	private MyComponentPoolSO _pool;

	private void Start()
	{
		_pool = ScriptableObject.CreateInstance<MyComponentPoolSO>();
		_pool.name = gameObject.name;
		_pool.Factory = ScriptableObject.CreateInstance<MyComponentFactorySO>();
		_pool.InitialPoolSize = _initialPoolSize;
		MyComponent myComponent = _pool.Request();
		//Do something with it
		_pool.Return(myComponent);
	}

}
```

### More info

To learn more about object pooling in general, check out the following links:

- [Intro to Object Pooling, on Unity Learn](https://learn.unity.com/tutorial/introduction-to-object-pooling)