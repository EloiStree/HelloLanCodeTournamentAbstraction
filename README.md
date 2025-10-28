> Why use this tool?

You want to create an offline coding tournament game where hundreds of players’ programs compete around a game engine running as the tournament server.

------------

# Hello LAN Code Tournament Abstraction

> Make some code tournament game in real time inside a LAN network to allow coders to learn outside of a library.

**! Under construction !**

Godot version: [https://github.com/EloiStree/2025_10_27_gdp_lan_code_tournament_abstraction](https://github.com/EloiStree/2025_10_27_gdp_lan_code_tournament_abstraction)   
Unity version: [https://github.com/EloiStree/2025_10_27_upm_lan_code_tournament_abstraction](https://github.com/EloiStree/2025_10_27_upm_lan_code_tournament_abstraction)  

The network part of the project is outside of this code:

* UDP: Fast, easy but not compatible on some platforms
  * Unity version: [https://github.com/EloiStree/2020_11_29_upm_udp_thread_in_out_gate](https://github.com/EloiStree/2020_11_29_upm_udp_thread_in_out_gate)
  * Godot version: [https://github.com/EloiStree/2025_10_27_gdp_udp_in_out_gate.git](https://github.com/EloiStree/2025_10_27_gdp_udp_in_out_gate.git)
* WebSocket: Works on most platforms but not on microcontrollers
  * Unity version: [https://github.com/EloiStree/2025_05_01_upm_trusted_websocket](https://github.com/EloiStree/2025_05_01_upm_trusted_websocket)
  * Godot version: [https://github.com/EloiStree/2025_10_28_gdp_trusted_websocket.git](https://github.com/EloiStree/2025_10_28_gdp_trusted_websocket.git)

---

# Concept

You want to create a coding tournament game?

Easy — you just need a bit of *in/out*.

* In: some input in IID format.
* Out: some bytes to give compressed high-frame information
* Out: some text to give game information
* Inside: some code to manage the input of 1–1024+ players.

You can use UDP or WebSocket depending on the context.
The code abstraction won’t have it; you have to add it to your project.

This toolbox is here to provide an easy drag-and-drop set of tools to design games for coders inside your engine.

I/we am going to provide a version in Unity3D and Godot.
Feel free to ping me if you want to code another engine version.

## In: IID

I am using the IID format.

Index, int: player-given network index
Value, int: integer that represents an action in your game
Date, ulong: timestamp representing when to do the action with NTP UTC

You can look at the format I created here:
[https://github.com/EloiStree/IID](https://github.com/EloiStree/IID)

And look at the Scratch to Warcraft standard I am using for my own project:
[https://github.com/EloiStree/2024_08_29_ScratchToWarcraft](https://github.com/EloiStree/2024_08_29_ScratchToWarcraft)

We just want to give them the ability to send input in an integer format.
Nothing more.
Hackers are going to hack around with more power.

## Timing: NTP

Network Time Protocol is a standard to synchronize the code of an application at the same time using a server.
You have some Wi-Fi routers that can have it, or install it on a Raspberry Pi.
If online, you can use an official server.

The idea is simple: check how many milliseconds of difference there are between your computer and the server and add it to your DateTime in your code.

To have an abstract layer, you have to code the NTP connection and give your game the number of milliseconds between the server and the game.

Having the same NTP time allows some crazy magic stuff when you think about it.
For example, you can sync full map events based on time to avoid sending them to all players through the network.

A code to run NTP on your Raspberry Pi:  
[https://github.com/EloiStree/2025_01_01_HelloPiOsNtpServer](https://github.com/EloiStree/2025_01_01_HelloPiOsNtpServer)    

# Out: byte array

## Players Position

When you have 256 players, you are going to need bytes and not text.

Type of Data (int) | Format Used (int) | Number of Players (int) | Index Element (int) | Size of Player Element (int) | All Players as Bytes

To differentiate player positions from the rest of the game info bytes,
I suggest using a sequence like 1, 2, 3 … 3, 2, 1 to represent player positions.

Be careful not to push data at 60 fps — you have to transport all that over Wi-Fi.

Outside of a LAN, you can’t update object positions in your game — too much data.
So for bullets, as an example, I tip you to provide size, start origin, speed, and timestamp NTP.

There are variants for the player position:  
* Id 0: Header | Position (Float 3) | Euler Rotation (Float 3) | Quaternion Rotation (Float 4) | Scale (float)
  * Full information of the player but 42 bytes per player
* Id 1: Header | Position (Float 3) | Euler Rotation (Float 3)
  * Minimum, no scaling, 18 bytes
* Id 2: Header | Position (Float 2) Map | Position (Float 2) World | Direction (float)
  * World of Warcraft kind, 20 bytes
* Id 3: Header | Position (byte 2) Map as percent 1–100 | Direction (byte)
  * World of Warcraft compressed in a color kind, 3 bytes
* Id 4: Header | Position mm (ushort 3) | Direction (byte)
  * If your game is only played in 65 meters with millimeter precision, 7 bytes
* Id 5: Header | Position cm (ushort 3) | Direction (byte)
  * If your game is only played in 650 meters with centimeter precision, 7 bytes

Example: a drone game with 256 players on 650-meter format:
12 + 7 * 256 = 1804 bytes or 1.8 KB

It means you have 18 KB per second at 10 frames per second,
and so 4608 KB per second to push or 4.8 MB.

On a LAN game it is okay, but on an online game, that can start to be expensive.

## Bullet Hell

Most of my games are going to use some bullet hell concepts.
You can’t update the position of 2000 bullets on the network every 10 frames.


So the idea is simple: you are notified of the appearance and destruction of a linear bullet.

To differentiate bullets from the rest of the game info bytes,
I suggest using a sequence like 3,2,1 … 1,2,3 to represent player positions.

Bullets:

* Start:
  * Server timestamp (ulong)
  * Pool ID (byte)
  * Bullet ID in the Pool ID (int)
  * Owner of the bullet ID (int)
  * Start Position (float 3)
  * Start Direction (float 3)
  * Diameter in meters (float)
  * Speed per second in mm (uint)
* End: depending on the game designer and collision on server
  * Server timestamp (ulong)
  * Pool ID (byte)
  * Bullet ID in the Pool ID (int)
  * End Position (float 3)

Warning: some game designs can lead to having 50,000 bullets.
In those games you will have to make a choice.

# Out: text

## Game Info

Players that are going to play your game are junior developers that want to learn.
If all the game info is stored in bytes format... they are going to hate you.

Provide them a manual of the text they can receive:

* "GAMESTART" and "GAMEOVER"
* "READY" and "GO"
* "WINNER IS: {PLAYER NAME}"
* "POS:X...:Y...:Z...:OBJECTIVE APPEARED"
* "POS:X...:Y...:Z...:AFK PLAYER DETECTED"
* "POS:X...:Y...:Z...:PLAYER IS NOW TOP 8"

---

# My goal here?

Basic?

* Have the ability to quickly prototype small tournaments every week.
* Allow the community to mod by creating their own code tournaments in their engine.

Why LAN?

* Because it is easier to master the security of offline tournaments.

Why Python on Pico W?

* Python is what’s used to teach kids how to code.
* I need a sandbox to isolate the code without hackers having fun around.
