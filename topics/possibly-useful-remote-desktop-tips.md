# Possibly useful remote desktop tips

I've worked in 2 companies so far that uses Windows Remote Desktop Protocol (RDP) to launch a development environemnt. I think it's neat, but the one caveat is that if we try to maximize the window it goes into full screen mode. It's not really an issue per se, but usually these environments within RDP are restricted such that you're not able to use the email client nor chat application.

It's a little cumbersome to switch out every time you need to do "socialise". By socialise I mean repsonding to emails and chat messages about "why does X not work". I know that developers are supposed to be introverts and only speak to rubber ducks, but I also don't wanna miss the free snacks at David's table because he just came back from a holiday to Japan.

Fortunately I've managed to find my way arounds this problem, so here's how it's done:

### You don't always have to start with full screen

When you launch RDP and select/type in your remote desktop address, you're actually given the opportunity to make some adjustments to suit your needs. You can reduce the amount of fancy data that's being trasmitted to possibly reduce input lag, but more importantl you're able to adjust the display at launch.

1. Click `Show Options` at the bottom-left of the RDP launch window
2. Select the `Display` tab
3. Adjust the slider to your desired resolution. Slider all the way to the right means launch at full screen. Slider all the way to the left means you're a masochist who enjoys self torture in tiny windows.

The "2nd highest" option is basically the window in your screen's native resolution, just not in full screen. Although part of your window will definitely get hidden by the taskbar or just be out of the screen due to the title bar taking up some pixels. Either way, as long as the slider doesn't say "Full Screen", you'll launch in window mode.

### Maximizing doesn't have to mean full screen

The Windows RDP works in a way that when you click on the "maximize" button (the square icon in between minizine and close), the window goes into full screen mode if the resolution that you set during launch is greater than your native resolution. Conversely, if you adjust it to a resolution that's SMALLER than your native resolution, it will only adjusts the content resolution to match the launch resolution and not go into full screen. This helps us to "maximize" the window without blocking the native desktop environment!

### Custom resolutions

The slider in the RDP lanch window only comes with a few pre-defined resolutions. These are usually in the ratio of your current display, i.e. if you're using a `16:9` display you will get options in a `16:9` format. It's not really an issue, but the avaible screen real estate on a `16:9` display is not in the `16:9` ratio. It' s a`16:<something less than 9>` ratio. That's because the taskbar and title bar also take up pixels. So mathematically, the height has to be `9/16 * <screen width> - <title bar height> - <taskbar height>`. I don't know for sure how windows is calculating the height of these 2 items, so I don't have a magic way to find it.

Anyway, if you do find it, you would want to launch the window in that exact resolution. The question is how? The display tab doesn't give us any way to do this. Well it's a good thing we know how to use a terminal!

```cmd
mstsc /w:<width in pixels> /h:<height in pixels>
```

You can now go into a **true** maximized window when using RDP. Giving you maximum screen real estate alongside full access to the native desktop.

### Saved states

Remember how I mentioned above that _as long as the slider doesn't say "Full Screen", you'll launch in window mode._?. That's not entirely true. Apparently RDP remembers the configuration that was done in the last session. So if you were maximized, the next time you open RDP would also be maximized! The same applies to resolutions. This means, if you launched RDP with a custom resolution, you will keep your native resolution until you accidentally adjust it.
