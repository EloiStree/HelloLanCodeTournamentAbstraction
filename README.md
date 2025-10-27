# Hello Lan Code Tournament Abstraction

Make some code tournament game in real time to allows coder to learn outside of library.  
   
Godot version: https://github.com/EloiStree/2025_10_27_gdp_lan_code_tournament_abstraction  
Unity Version: https://github.com/EloiStree/2025_10_27_upm_lan_code_tournament_abstraction  
  
The network part of the project is outside of this code:  
- UDP: Fast easy but not compatible on some platform
  - Unity Version: (To Add)
  - Godot Version: (To Add)
- Websocket: Work on most platform but not on Micro-Controller
  - Unity Version: (To Add)
  - Godot Version: (To Add)


 ---------------

You want to create a coding tournament game ?

Easy you just need a bit of in/out.
  - In: some input in IID format.
  - Out: Some byte to give compress high frame information
  - Out: Some text to give game information
  - Inside: Some code to manage the input of 1-1024+ players.

You can use UDP or Websocket depending on the context.
The code abstraction won't have it, you have to add it in your project.

This toolbox is here to provide a easy drag and drog of tools to design game for coder inside your engine.
I, Eloi Stree, am going to provide a version in Unity3D and Godot.
Feel free to ping me if you want to code an other engine version.

## In: IID

I am using the IID format.

Index,int: player given network index
Value,int: integer that representa action in your game
Date, ulong: timestramp representing when to do the action with NTP UTC

You can look at the format I created here:  
[https://github.com/EloiStree/IID](https://github.com/EloiStree/IID)

And look at the Scratch to Warcraft standard I am using for my own project:  
[https://github.com/EloiStree/2024_08_29_ScratchToWarcraft](https://github.com/EloiStree/2024_08_29_ScratchToWarcraft)

## Timing: NTP

Network Time Protocole is a standard to synchronise code of an application at the same time using a server.
You have some Wifi Routeur that can have it or install it on Raspberry Pi.
If online you can use a official server.

The idea is simple: Check what number of milliseconds of different there is between your computer and the server and add it to your DateTime in your code.

To have an abstract layer, you  have to code the NTP connection and give your game the number of milliseconds between the server and the game.

Having the same NTP time allows some crazy magic shit when you think a bit about it.
For example, you can sync the full map events based on time to avoid to send it to all the player through network.


# Out: byte array

## Players Position

When you have 256 players... You are going to need bytes and not text.

So push Type of Data (int) | Format Used (int) | Number Of Player (int) | All players as bytes

Be careful to not push 60 fps the data, you have to transport all that in the Wifi.

Outside of a LAN logic you can update position of object in your game... To much data.
So for bullets, I tip your to provide size, start origine, speed and timestramp ntp

They are variant;
- Header | Position (Float 3) | Euler Rotation (Float 3)| Quaternino Rotation (Float 4) | Scale (float)
  - Full information of the player but 42 bytes per player  
- Header | Position (Float 3) | Euler Rotation (Float 3)
  - Minimium not scaling 18 Bytes  
- Header | Position (Float 3) | Direction (byte)
  - World of warcraft kind 13 bytes 
- Header | Position mm (ushort 3) | Direction (byte)
  - If your game is only played in 65 meter with millimeter precision 7 bytes 
- Header | Position cm (ushort 3) | Direction (byte)
  - If your game is only played in 650 meter with centimeter precision 7 bytes

Example a drone game with 256 player on 650 meter format:
12 + 7 * 256= 1804 bytes or 1.8 ko
It means you have 18 ko per seconds at 10 frame per seconds
And so 4608 ko per seconds do to push or 4.8 Mo

On a LAN game it is ok, but on a online game, that can start to be expensive.

## Bullet Hell

Most of my game are going to use some bullet hell concept.
You can t update the position of 2000 bullets on the network every 10 frames.

So the idea is simple you are notified of the apparition and destruction of a linear bullet.

Bullets:
- Start:
  - Start Position (float 3)
  - Start Quaternion (float 4)
  - Start Euler Rotation (float 3)
  - Diameter in meter (float)
  - Server timestamp (ulong)
  - Speed per second in mm (uint)
  - Owner of the bullet id (int)
  - Pool ID (byte)
  - Bullet ID in the Pool ID (byte)
- End: Depending of the game designer and collision on server
  - End Position (float 3)
  - Server Timestamp (ulong)

Warning some game design can lead to have 50.000 bullets
In those game you will have to make a choose.

# Out: text

## Game Info

Player that are going to player your game a junior developer that want to learn.
If all the game infor is store in bytes format... They are going to hate you.

Provide them a manual of the text they can receive:
- "GAMESTART" and "GAMEOVER" 
- "READY" and "GO"
- "WINNER IS: {PLAYER NAME}"
- "POS:X...:Y...:Z...:OBJECTIF APPEART
- "POS:X...:Y...:Z...:AFK PLAYER DETECTED"
- "POS:X...:Y...:Z...:PLAYER IS NOW TOP 8"


# To complete




The idea of this file is to list outside of UPD Websocket the tranfict to support.
To make an abstract layer for the code tournament we want to organise and lets the community mod around.
  


