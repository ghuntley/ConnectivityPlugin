# Getting Started

## Setup
* NuGet: [Xam.Plugin.Connectivity](http://www.nuget.org/packages/Xam.Plugin.Connectivity) [![NuGet](https://img.shields.io/nuget/v/Xam.Plugin.Connectivity.svg?label=NuGet)](https://www.nuget.org/packages/Xam.Plugin.Connectivity/)
* `PM> Install-Package Xam.Plugin.Connectivity`
* Install into ALL of your projects, include client projects.


## Using Connectivity APIs
It is drop dead simple to gain access to the Connectivity APIs in any project. All you need to do is get a reference to the current instance of IConnectivity via `CrossConnectivity.Current`:

```csharp
public bool DoIHaveInternet()
{
    return CrossConnectivity.Current.IsConnected;
}
```

There may be instances where you install a plugin into a platform that it isn't supported yet. This means you will have access to the interface, but no implementation exists. You can make a simple check before calling any API to see if it is supported on the platform where the code is running. This if nifty when unit testing:

```csharp
public bool DoIHaveInternet()
{
    if(!CrossConnectivity.IsSupported)
        return true;

    return CrossConnectivity.Current.IsConnected;
}
```

## Disposing of Connectivity Plugin
This plugin also implements IDisposable on all implementations. This ensure that all events are unregistered from the platform. Only dispose when you need to and are no longer listening to events. The next time you gain access to the `CrossConnectivity.Current` a new instance will be created.

```csharp
public bool DoIHaveInternet()
{
    if(!CrossConnectivity.IsSupported)
        return true;
        
    //Do this only if you need to and aren't listening to any other events as they will not fire.
    using(var connectivity = CrossConnectivity.Current)
    {
        return connectivity.IsConnected;
    }
}
```


## Permissions & Additional Setup Considerations

### Android:
The `ACCESS_NETWORK_STATE` and `ACCESS_WIFI_STATE` permissions are required and are automatically added to your Android Manifest when you compile. No need to add them manually!

By adding these permissions [Google Play will automatically filter out devices](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions-features) without specific hardware. You can get around this by adding the following to your AssemblyInfo.cs file in your Android project:

```csharp
[assembly: UsesFeature("android.hardware.wifi", Required = false)]
```

## Architecture

### What's with this .Current Global Variable? Why can't I use $FAVORITE_IOC_LIBARY
You totally can! Every plugin I create is based on an interface. The static singleton just gives you a super simple way of gaining access to the platform implementation. Realize that the implementation of the plugin lives in your iOS, Android, Windows, etc. Thies means you will need to register it there by instantiating a `CrossConnectivityImplementation` from the platform specific projects.

If you are using a ViewModel/IOC approach your code may look like:

```csharp
public MyViewModel()
{
    readonly IConnectivity connectivity;
    public MyViewModel(IConnectivity connectivity)
    {
        this.connectivity = connectivity;
    }

    public bool IsConnected => connectivity?.IsConnected ?? false;
}
```

### What About Unit Testing?
To learn about unit testing strategies be sure to read my blog: [Unit Testing Plugins for Xamarin](http://motzcod.es/post/159267241302/unit-testing-plugins-for-xamarin)


<= Back to [Table of Contents](README.md)