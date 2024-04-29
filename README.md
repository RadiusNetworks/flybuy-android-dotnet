# Flybuy SDK for Android on .Net

## Setup

The Flybuy SDK is published on nuget.org at the following location: https://www.nuget.org/packages/Radiusnetworks.Flybuy.Android

### Add the Flybuy SDK to your .Net MAUI Project for Android

Select `Packages` in your VS project and go to `Manage NuGet Packages...`. Search for `Radiusnetworks.Flybuy.Android` and click `Add Package`. The latest SDK will be added to your project and can be referenced with the following:

```csharp
using Flybuy;
using Flybuy.Data;
```

### Initialize SDK on Launch

The Andorid SDK consists of two modules:

 * pickup
 * notify

Flybuy needs to be setup and configured at application launch. However, it does not run in the background or use device resources until there is an active order.

The easiest way to configure Flybuy in your app is to extend the `FlyBuyApplication` and call the `FlyBuyCore.configure(...)` method from `onCreate()` to setup the auth token. Also, any modules that are used need to be configured. The example below shows all modules, but make sure to only configure the modules needed.

```csharp
[Application]
public class MyApplication : FlyBuyApplication
{
    public MyApplication(IntPtr javaReference, Android.Runtime.JniHandleOwnership transfer) : base(javaReference, transfer)
    {
    }

    public override void OnCreate()
    {
        base.OnCreate();

        string appToken = "nnn.xxx"

        var builder = new ConfigOptions.Builder(appToken);
        builder.SetDeferredLocationTrackingEnabled(true);
        Core.Configure(this, builder.Build());

        var Pickup = PickupManager.Manager.GetInstance(null) as PickupManager;
        Pickup.Configure(this);
        //if using the Notify module
        var Notify = NotifyManager.Manager.GetInstance(null) as NotifyManager;
        Notify.Configure(this);
    }
}
```

The modules also provide methods for updating permissions. The following example handles location permission updates.

```csharp
private const int FLYBUY_PERMISSIONS_REQUEST = 42;

private void RequestLocationPermissions() {
    var permissions = CheckForMissingLocationPermissions();
    if (permissions.Count > 0) {
        ActivityCompat.RequestPermissions(this, permissions.ToArray(), FLYBUY_PERMISSIONS_REQUEST);
    } else {
        var Pickup = PickupManager.Manager.GetInstance(null) as PickupManager;
        Pickup.OnLocationPermissionChanged();
    }
}

public override void OnRequestPermissionsResult(int requestCode, string[] permissions, [GeneratedEnum] Permission[] grantResults)
{
    if(requestCode == FLYBUY_PERMISSIONS_REQUEST) {
        var Pickup = PickupManager.Manager.GetInstance(null) as PickupManager;
        Pickup.OnLocationPermissionChanged();
    }
}
```

## Components

The following section provides some examples C# syntax for the customer and order components. Refer to the [native documentation](https://www.radiusnetworks.com/developers/flybuy/#/sdk-2.0/customer) for complete details on the SDK components.

Use the following to determine if a customer is associated with the app:
```csharp
Customer currentCustomer = Core.customer.Current;
```

If no customer, then create the customer using:
```csharp
CustomerInfo customerInfo = new CustomerInfo(
        "Marty McFly",
        "DeLorean",
        "Silver",
        "OUTATIME",
        "555-555-5555"
);
Core.customer.Create(customerInfo, true, true, null, null, createCustomerCallback);
```

To claim an order use:
```csharp
Core.orders.Claim(redemptionCode, customerInfo, null, claimCallback);
```

To update an order use:
```csharp
Core.orders.UpdateCustomerState(currentOrder.Id, CustomerState.Completed, completeOrderCallback);
```

