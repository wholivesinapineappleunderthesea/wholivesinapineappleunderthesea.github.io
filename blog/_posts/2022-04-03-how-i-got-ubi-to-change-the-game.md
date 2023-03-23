---
layout: post
title: How I Got Ubi To Fix A RCE Vulnerability 
---

Siege uses a very interesting packet/message system. In many other games there are simply entity indexes that are used inside networked messages to describe targets in a packet. Siege uses object ids. These object ids are the same on client and server and it is very easy to send encode this for a packet.

The issue is, they never checked what the object is. It just took the described object and casted it. No checks were ever done to make sure object x is actually of the type it is referenced as.

I knew this was an issue for a while, but I never really thought it would be too useful because its VERY undefined behaviour. You would need things to line up perfectly on both client and server build configs for it to be something interesting to exploit.

However I knew there would be one way: ladders. 

Back in 2020 cheaters were using ladders to fly around the map. You could attach to a ladder however far you were away from it, and move anywhere on the vertical axis at any speed. This was patched by adding a check on distance (it was around two metres) and a check to make sure you dont move above the top of the ladder.

The message sent from the client to attach to a ladder sends the ladder's object id to the server, where it is decoded and casted. Then a check on 2d distance is done by going into the ladder component, getting its entity and checking its position against the player's.

I knew this could be abused, there was no check to make sure that it was actually a ladder. It could be ANYTHING... I tried it with my local player's component and bam it works, I get attached to an imaginary ladder right at my feet, the animation plays and I fall off it a few metres up.

<video width="640" height="348" controls="controls">
<source src="https://cdn.discordapp.com/attachments/767867719151648818/961076809309491280/a2022-01-12_18-00-05.mp4" type="video/mp4">
</video>

This was the result, let me explain:

I am on a ladder, server side.

Due to the structural layout of the Pawn class (the object I'm treating as a ladder) this "ladder" has zero height, so once the attach animation is over I will be instantly kicked off of the "ladder".

It is moving me up along with the animation, you don't see it here but on the server and on other clients they see me sliding up an invisible ladder.

This is exactly what you don't want to happen with your video game. There is a pointer that I have arbitrary control over... 

Now this isn't very exciting on its own, I move slow and not very far, but it's a start!

This is where the second problem with ladders come into play:

If you switch a ladder while attached to it, the validation COMPLETELY breaks. You are given complete control of your position in the vertical axis.

You can move up down thru objects, anything.

This was the result:

<video width="640" height="348" controls="controls">
<source src="https://cdn.discordapp.com/attachments/902731816115527703/935395408648224828/2022-01-15_14-52-12.mp4" type="video/mp4">
</video>


This was first used January 12th, and was patched a month later on February 10th.

![feb 10th update](assets/ubifix.png "February 10th Update")

Ubi added a check to make sure that before casting, it is infact the correct object type.