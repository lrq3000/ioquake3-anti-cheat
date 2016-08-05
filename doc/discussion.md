Here is a compilation of snippets of discussion with the author, Laszlo:

> The anti-wallhack module does work, but it has a big problem: loss of sound direction (inherent to the mechanism used, i.e. repositioning of clients). I have been advised by experienced Q3 players that it is not acceptable, I have some ideas how to fix this but it needs a major redesign of the way Q3A handles client events. Right now I don't have the time to deal with it, but I will soon resume working of the anti-wallhack module. I will let you know when I have a correctly working version.
>
> The problem is that most of the player sound events are tied to players (in the original code the position of sound is derived form that of the player who generates the sound). I think it is possible to send the sound events separately, but that requires a modification of the event handling code of Q3A. Also a cheater could still deduce from the sound event that somebody is at that position, but would not know the identity of the player. That's the best we can achieve, unless we do not play the sounds at all (but it is no acceptable). I will let you know as soon as I have something useful.
>

Also note that it was implemented in q3reloaded and in ET:Legacy ([link1](https://github.com/etlegacy/etlegacy/blob/master/docs/game/anticheat.html), [link2](https://yuki.la/v/167831601#p167833932)).