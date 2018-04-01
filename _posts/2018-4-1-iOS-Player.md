---
layout: post
title: SESHapp stories #1
---

### Pre-Story
One night my friend asked me to develop an app for TeamSESH.

### Story
It's been pretty long time since I started developing SESHapp and faced a lot of interested things making a stream player, but this post is not about that. The new update would include a lot of different things: news, push notifications, team tab, tours tab, **offline player**, live stream player, bugfixes. I would focus on offline player in this post. Why? Since there's not that much info in google. I mean, there is some useful information, but mostly those are just pointers to libs or Apple docs. The first problem I faced was the flow control and architecture.

### Architecture & Flow control
About a month ago my colleague offered me to try ReactiveX and I didnt like it, even tho I used KVO before and RX looked kinda the same, except _syntax was, uhm, idk how to explain that._ 

First of all, we have two screens:
- The common: tracks list and artists
- The main one: player.

Both have to interact the same. If we change track from the common one, the track _must_ be changed in the main one. The first thing that came out to my head was using KVO and tranfer some important pointers of the current playing item and store track info somewhere in shared memory (_can I say like that?_). But this idea is pretty old. In case of fashion. Who uses system's KVO when we have RX? Hello, RxCocoa. My thoughts were: huh, it's going to be pretty easy and I'll write my own wrappers. But nah, I found a pretty cool extension for AVPlayer: https://github.com/pmick/RxAVFoundation. It made the idea much more easier to implement. Thanks, based God. 

>So, the way is: subscribe to main events (player's status, such as play, pause, track change) in the common screen, post them to both screens, implement some iOS delegates, and, bahm, its working. Easy. 

Doing all of this in ViewController isn't the best method. Let's create PlayerHelper that would control the player itself, track change, player's status, receiving remote controls from system. Done. Now we have a pretty cool wrapper for AVPlayer, that would be easily controlled via shared instance. Everything works, except that our track does not change whenever the track did reach end. Let's make a enum that simply tell us what's going on on player's side:

>case Paused = 0

>case Playing = 1

>case WaitingForNext = 2

>case WaitingForPrev = 3

If the player reached the end, then simply change our player's internal status to _WaitingForNext_ and let common view controller change the track. Simple as that.

Conclusion: two screens, first one controls mostly everything (events, track change, tracks pagination, etc), the second one controls the player itself and delegates to the first one all the events. The first one uses dumb stuff like _updatePlayer_, _updateCurrentTime_ that are functions of the Player itself. 

### But then....
Volume. Seems easy, yay? Yes, it is, but I dont like to install raw app's builds to my iPhone and mostly using the simulator. I googled a lil bit and found out that I have to use _MPVolumeView_ to change system volume.

WTF? Like, really, implementing a view to change system's volume? Ok, let's assume thats about _Think Different_. Some SO post told me that _MPVolumeView_ is a subclass of _UISlider_. Which is not. It looks like slider, but it's not. It's a subclass of UIView (at least, in terms of Interface Builder). Changing background color to clear, implementing subclass - phew - not working. Kek. Why? Well, for some reason iOS simulator does not support displaying _MPVolumeView_. It took me some times to realize that.


### Conclusion
Making simple audio player is not a big deal with all the stuff we have atm, but some problems are still there. Next time I'll tell about caching avplayer, since I have no idea how to implement this atm.