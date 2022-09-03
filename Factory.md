![Factory Diagram](https://github.com/UnityTechnologies/open-project-1/blob/main/Docs/WikiImages/Factory_diagram.png)

Without some sort of abstraction, generic systems required to handle a variety of types must implement the creation of each type independently, leading to a lot of boilerplate code. This can be the case when you have to `new` many non-abstract types (such as Customer, User, etc.), `Instantiate` MonoBehaviours as components on a GameObject, or `CreateInstance` of ScriptableObjects. The abstract **Factory pattern** allows for the creation of various types without the knowledge of their specific creation implementation.

In this game for instance, we use it a lot in our [Object Pooling](https://github.com/UnityTechnologies/open-project-1/wiki/Object-Pooling) solution.

## Table of Contents
- [New-able type with an empty constructor](#new-able-type-with-an-empty-constructor)
- [New-able type with constructor arguments](#new-able-type-with-constructor-arguments)
- [Components](#components)
- [Components on prefabs](#components-on-prefabs)
- [ScriptableObjects](#scriptableobjects)
- [More info](#more-info)

### New-able type with an empty constructor

```cs
using UnityEngine;
using UOP1.Factory;

[CreateAssetMenu(fileName = "NewCustomerFactory",menuName="Factory/Customer")]
public class CustomerFactorySO : FactorySO<Customer>
{
    public override Customer Create(){
        return new Customer();
    }
}
```

> **Note:** The CreateAssetMenu attribute may be omitted if the factory is only created via CreateInstance at runtime.

### New-able type with constructor arguments

```cs
using UnityEngine;
using UOP1.Factory;

[CreateAssetMenu(fileName = "NewCustomerFactory",menuName="Factory/Customer")]
public class CustomerFactorySO : FactorySO<Customer>
{
    [SerializeField]
    private int _name;

    public int Name{
        set{
            _name = value;
        }
    }

    public override Customer Create(){
        return new Customer(_name);
    }
}
```

> **Note:** The public property setter may be omitted if the factory is only created as an asset in the project.

### Components

```cs
using UnityEngine;
using UOP1.Factory;

[CreateAssetMenu(fileName = "NewAudioSourceFactory",menuName="Factory/AudioSource")]
public class AudioSourceFactorySO : FactorySO<AudioSource>
{
    public override AudioSource Create(){
        return new GameObject("AudioSource").AddComponent<AudioSource>();
    }
}
```

### Components on prefabs

```cs
using UnityEngine;
using UOP1.Factory;

[CreateAssetMenu(fileName = "NewAudioSourceFactory",menuName="Factory/AudioSource")]
public class AudioSourceFactorySO : FactorySO<AudioSource>
{
    [SerializeField]
    private AudioSource _prefab;

    public int Prefab{
        set{
            _prefab = value;
        }
    }

    public override AudioSource Create(){
        return Instantiate(_prefab);
    }
}
```

### ScriptableObjects

```cs
using UnityEngine;
using UOP1.Factory;

[CreateAssetMenu(fileName = "NewConfigFactory",menuName="Factory/Config")]
public class ConfigFactorySO : FactorySO<ConfigSO>
{
    public override ConfigSO Create(){
        return ScriptableObject.CreateInstance<ConfigSO>();
    }
}
```

### More info
- [Factory method pattern, on Wikipedia](https://en.wikipedia.org/wiki/Factory_method_pattern)