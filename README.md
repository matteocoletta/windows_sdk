## Summary

This is the Windows SDK of adjust™. You can read more about adjust™ at [adjust.com](http://adjust.com).

## Table of contents

* [Example app](#example-app)
* [Basic integration](#basic-integration)
   * [1 - Install the package Adjust using NuGet Package Manager](#install-adjust-package)
   * [2 - Integrate adjust into your app](#integrate-adjust-package)
   * [3 - Update adjust settings](#update-adjust-settings)
      * [App Token & Environment](#app-token-and-environment)
      * [Adjust Logging](#adjust-logging)
   * [4 - Build your app](#build-your-app)
* [Additional features](#additional-features)
   * [1 - Add tracking of custom events](#custom-events-tracking)
   * [2 - Add callback parameters](#add-callback-params)
   * [3 - Partner parameters](#partner-params)
   * [4 - Add tracking of revenue](#revenue-tracking)
   * [5 - Set up deep link reattributions](#deeplink-reattributions)
   * [6 - Enable event bufferings](#enable-event-buffering)
   * [7 - Set listener for attribution changes](#attribution-changes-listener)
* [Troubleshooting](#troubleshooting)
   * TODO
* [License](#license)

## <a id="example-app"></a>Example app

There are different example apps inside the [`Adjust` directory][example]: 
1. `AdjustUAP10Example` for Universal Windows Apps,
2. `AdjustWP81Example` for Windows Phone 8.1,
3. `AdjustWSExample` for Windows Store. 

You can use these example projects to see how the adjust SDK can be integrated into your app.

## <a id="basic-integration"></a>Basic Integration

These are the basic steps required to integrate the adjust SDK into your
Windows Phone or Windows Store project. We are going to assume that you use
Visual Studio 2015 or later, with the latest NuGet package manager installed. A previous version that supports Windows Phone 8.1 or Windows 8 should also work. The
screenshots show the integration process for a Windows Universal app, but the
procedure is very similar for both Windows Store or Phone apps. Any differences with Windows Phone 8.1
or Windows Store apps will be noted throughout the walkthrough.

### <a id="install-adjust-package"></a>1. Install the package Adjust using NuGet Package Manager

Right click on the project in the Solution Explorer, then click on `Manage NuGet Packages...`.
In the NuGet Package Manager window, click on "Browse" tab, enter "adjust" in the search box, and press Enter.
Adjust package sould be the first search result, click on it, and in the right pane, click on Install.

![][adjust_nuget_pm]

Another method to install Adjust package is using Package Manager Console.
In the Visual Studio menu, select `TOOLS → NuGet Package Manager → Package
Manager Console` (or, in older version of Visual Studio `TOOLS → Library Package Manager → Package
Manager Console`) to open the Package Manager Console view.

After the `PM>` prompt, enter the following line and press `<Enter>` to install
the [Adjust package][NuGet]:

```
Install-Package Adjust
```

It's also possible to install the Adjust package through the NuGet Package
Manager for your Windows Phone or Windows Store project.

### <a id="integrate-adjust-package"></a>2. Integrate adjust into your app

In the Solution Explorer, open the file `App.xaml.cs`. Add the `using
AdjustSdk;` statement at the top of the file.

Here is a snippet of the code that has to be added in `OnLaunched` method of your app.

```cs
using AdjustSdk;

sealed partial class App : Application
{
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        var config = new AdjustConfig(appToken, environment);
        Adjust.ApplicationLaunching(config);
        // ...
    }
}
```

### <a id="update-adjust-settings"></a>3. Update adjust settings

#### <a id="app-token-and-environment"></a>App Token & Environment

Replace the `{YourAppToken}` placeholder with your App Token, which you can find in
your [dashboard].

Depending on whether or not you build your app for testing or for production,
you will need to set the `environment` parameter with one of these values:

```cs
string environment = AdjustConfig.EnvironmentSandbox;
string environment = AdjustConfig.EnvironmentProduction;
```

**Important:** This value should be set to `AdjustConfig.EnvironmentSandbox`
if and only if you or someone else is testing your app. Make sure to set the
environment to `AdjustConfig.EnvironmentProduction` just before you publish
your app. Set it back to `AdjustConfig.EnvironmentSandbox` if you start
developing and testing it again.

We use this environment to distinguish between real traffic and test traffic
from test devices. It is very important that you keep this value meaningful at
all times, especially if you are tracking revenue.

#### <a id="adjust-logging"></a>Adjust Logging

To see the compiled logs from our library in `released` mode, it is
necessary to redirect the log output to your app while it's being tested in `debug` mode.

To do this, use the `AdjustConfig` constructor with 4 parameters, where 3rd parameter is the
delegate method which handles the logging, and 4th parameter being Log Level:

```cs
// ....
protected override void OnLaunched(LaunchActivatedEventArgs e)
{
    string appToken = "hmqwpvspxnuo";
    string environment = AdjustConfig.EnvironmentSandbox;
    var config = new AdjustConfig(appToken, environment,
        msg => System.Diagnostics.Debug.WriteLine(msg), LogLevel.Verbose);
    // ...
}
// ....
```

You can increase or decrease the amount of logs you see in tests by setting the
4th argument of the `AdjustConfig` constructor, `logLevel`, with one of the following values:

```cs
logLevel: LogLevel.Verbose  // enable all logging
logLevel: LogLevel.Debug    // enable more logging
logLevel: LogLevel.Info     // the default
logLevel: LogLevel.Warn     // disable info logging
logLevel: LogLevel.Error    // disable warnings as well
logLevel: LogLevel.Assert   // disable errors as well
logLevel: LogLevel.Suppress // disable all logs
```

### <a id="build-your-app"></a>4. Build your app

From the menu, select `DEBUG → Start Debugging`. After the app launches, you
should see the Adjust debug logs in the Output view. Every Adjust specific log
starts with ```[Adjust]``` tag, like in the picture below:

![][debug_output_window]

## <a id="additional-features"></a>Additional features

Once you have integrated the adjust SDK into your project, you can take
advantage of the following features.

### <a id="custom-events-tracking"></a>1. Add tracking of custom events

You can use adjust to track any event in your app. Suppose you want to track
every tap of a button. You would have to create a new event token in your
[dashboard]. Let's say that event token is `abc123`. In your button's `Button_Click`
method, you can add the following lines to track the click:

```cs
var adjustEvent = new AdjustEvent("abc123");
Adjust.TrackEvent(adjustEvent);
```

The event instance can be used to further configure before you begin tracking.

### <a id="add-callback-params"></a>2. Add callback parameters

You can register a callback URL for the events in your [dashboard]. We will send a GET request to this URL whenever an event is tracked. You can also add callback parameters to the event by calling `AddCallbackParameter` on the
event instance before tracking it. We will then append these parameters to your specified callback URL.

For example, suppose you have registered the URL
`http://www.adjust.com/callback` then track an event like this:

```cs
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.AddCallbackParameter("key", "value");
adjustEvent.AddCallbackParameter("foo", "bar");

Adjust.TrackEvent(adjustEvent);
```

In that case we would track the event and send a request to:

```
http://www.adjust.com/callback?key=value&foo=bar
```

We should mention that we support a variety of placeholders like
`{win_adid}` that are usuable as parameter values. In the resulting callback,
this placeholder would be replaced with the Windows Advertising ID of the current device.
You should also note that we do not store any of your custom parameters, but only append
them to your callbacks. If you haven't registered a callback for an event,
these parameters won't even be read.

You can read more about using URL callbacks, including a full list of available
values, in our [callbacks guide][callbacks-guide].

### <a id="partner-params"></a>3. Partner parameters

You can also add parameters to be transmitted to network partners for
integrations that have been activated in your adjust dashboard.

This works similarly to the callback parameters mentioned above, but these are instead
added by calling the `AddPartnerParameter` method on your `AdjustEvent` instance.

```cs
var adjustEvent = new AdjustEvent("abc123");

adjustEvent.AddPartnerParameter("key", "value");
adjustEvent.AddPartnerParameter("foo", "bar");

Adjust.TrackEvent(adjustEvent);
```

You can read more about special partners and these integrations in our [guide
to special partners.][special-partners]

### <a id="revenue-tracking"></a>4. Add tracking of revenue

If your users generate revenue by tapping on advertisements or making
in-app purchases, then you can track those revenues with events. Let's say a tap is
worth €0.01. You can then track the revenue event like this:

```cs
var adjustEvent = new AdjustEvent("abc123");
adjustEvent.SetRevenue(0.01, "EUR");
Adjust.TrackEvent(adjustEvent);
```

This can be combined with callback parameters, of course.

When you set a currency token, adjust will automatically convert the incoming revenue into the reporting revenue of your choice. Read more about [currency conversion here.][currency-conversion]

You can read more about revenue and event tracking in the [event tracking
guide.][event-tracking]

### <a id="deeplink-reattributions"></a>5. Set up deep link reattributions

You can set up the adjust SDK to handle any deep links (also known as URI activation in Universal apps) used to open your app. We will only read adjust-specific parameters. This is essential if you are planning to run retargeting or re-engagement campaigns with deep links.

In the `OnActivated` method of your app, call the method `AppWillOpenUrl`.

```cs
using AdjustSdk;

public partial class App : Application
{
    protected override void OnActivated(IActivatedEventArgs args)
    {
        if (args.Kind == ActivationKind.Protocol)
        {
            var eventArgs = args as ProtocolActivatedEventArgs;

            if (eventArgs != null)
            {
                Adjust.AppWillOpenUrl(eventArgs.Uri);
            }
        }
        //...
    }
}
```

### <a id="enable-event-buffering"></a>6. Enable event buffering

If your app makes heavy use of event tracking, you may want to delay some
HTTP requests in order to send them in a single batch per minute. You can enable
event buffering with your `AdjustConfig` instance:

```cs
var config = new AdjustConfig(appToken, environment);

config.EventBufferingEnabled = true;

Adjust.ApplicationLaunching(config);
```

### <a id="attribution-changes-listener"></a>7. Set listener for attribution changes

You can register a listener to be notified of tracker attribution changes. Due
to the different sources considered for attribution, this information cannot
be provided synchronously. The simplest way is to create a single anonymous
listener.

Please make sure to consider our [applicable attribution data
policies][attribution-data].

With the `AdjustConfig` instance, before starting the SDK, set the
`AttributionChanged` delegate with the `Action<AdjustAttribution>` signature.

```cs
var config = new AdjustConfig(appToken, environment);

config.AttributionChanged = (attribution) => 
    System.Diagnostics.Debug.WriteLine("attribution: " + attribution);
    
Adjust.ApplicationLaunching(config);
```

Alternatively, you could implement the `AttributionChanged` delegate
interface in your `Application` class and set it as a delegate:

```cs
var config = new AdjustConfig(appToken, environment);
config.AttributionChanged = AdjustAttributionChanged;
Adjust.ApplicationLaunching(config);

private void AdjustAttributionChanged(AdjustAttribution attribution) 
{
    //...
}
```

The delegate function will be called when the SDK receives the final attribution
information. Within the listener function you have access to the `attribution`
parameter. Here is a quick summary of its properties:

- `string TrackerToken` the tracker token of the current install.
- `string TrackerName` the tracker name of the current install.
- `string Network` the network grouping level of the current install.
- `string Campaign` the campaign grouping level of the current install.
- `string Adgroup` the ad group grouping level of the current install.
- `string Creative` the creative grouping level of the current install.
- `string ClickLabel` the click label of the current install.
- `string Adid` the ADID of the current install.

## <a id="troubleshooting">Troubleshooting

TODO

[adjust.com]: http://www.adjust.com
[dashboard]: http://www.adjust.com
[nuget]: http://nuget.org/packages/Adjust
[nuget_click]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/01_nuget_console_click.png
[adjust_nuget_pm]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/v4_12/adjust_nuget_pm.png
[wp_capabilities]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/03_windows_phone_capabilities.png
[wp_app_integration]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/04_wp_app_integration.png
[ws_app_integration]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/05_ws_app_integration.png
[debug_output_window]: https://raw.github.com/adjust/adjust_sdk/master/Resources/windows/v4_12/debug_output_window.png
[attribution-data]: https://github.com/adjust/sdks/blob/master/doc/attribution-data.md

[dashboard]:     http://adjust.com
[releases]:      https://github.com/adjust/adjust_android_sdk/releases
[import_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/01_import_module.png
[select_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/02_select_module.png
[imported_module]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/03_imported_module.png
[gradle_adjust]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/04_gradle_adjust.png
[gradle_gps]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/05_gradle_gps.png
[manifest_gps]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/06_manifest_gps.png
[manifest_permissions]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/07_manifest_permissions.png
[proguard]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/08_proguard.png
[receiver]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/09_receiver.png
[application_class]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/11_application_class.png
[manifest_application]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/12_manifest_application.png
[application_config]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/13_application_config.png
[activity]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/14_activity.png
[log_message]: https://raw.github.com/adjust/sdks/master/Resources/android/v4/15_log_message.png

[callbacks-guide]:      https://docs.adjust.com/en/callbacks
[event-tracking]:       https://docs.adjust.com/en/event-tracking
[special-partners]:     https://docs.adjust.com/en/special-partners
[example]:              https://github.com/adjust/windows_sdk/tree/master/Adjust
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies


## <a id="license"></a>License

The adjust SDK is licensed under the MIT License.

Copyright (c) 2012-2017 adjust GmbH,
http://www.adjust.com

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

