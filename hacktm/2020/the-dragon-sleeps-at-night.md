# The dragon sleeps at night

### Description
```
I made this console based dragon RPG.
Go kill the beast!

nc 138.68.67.161 60004
```

### Walkthrough
So we have to kill the dragon. There are just three problems:

1. We can only kill the dragon during the night. If he is awake, he will kill us instantly.
2. It would take forever to make enough money to buy the Level 5 sword. ($1 billion cost, and for each time we work, we can only enter a 3-digit number of hours at $1/hour).
3. Even the Level 5 sword is not powerful enough to kill the dragon

The first problem is simple enough: we just kill time until at least 18:00 before we visit him. But how do we afford a sword powerful enough to kill him?

After working a shift, our boss asks us how many hours we've worked, and pays us $1 per hour worked. He is also very trusting, so we can enter any value we like, but it must be no larger than three digits. $999 per shift will obviously not net us much money quickly, since the game has built in many delays.

But what if we try to trick the game itself? When asked how many hours we've worked, enter `9e9`. And voila, we get $9000000000. $9 billion. Nice!

```
-------------------------------
Boss wants to know how many hours you worked: > 9e9
9000000000.0 hours at $1/hour? That's $9000000000.0.
$9000000000.0 received.
-------------------------------
```


So we head to the store and buy a nice, shiny Level 5 sword. But if we wait until nighttime and visit the dragon now, we see it's still not enough! We need a Level 6 sword...

```
-------------------------------
Welcome to the dragon's cave
-------------------------------
You see the dragon sleeping next to a pile of bodies.
They look disturbingly fresh.
Carrying your glorious level 5 sword, you slowly walk over.
Carefully, you position the mighty weapon exactly over his skull.
BOOM! Perfect hit!
The dragon wakes up! He's not dead?
I was told level 5 would be enough!
'if only there was a level 6 sword' are your last thoughts...
...as the dragon obliterates you with a hurricane of fire.
Game over
```

But the store doesn't carry a Level 6 sword. Even trying to enter `6` when buying a sword gets us nowhere.

If we head to storage, we notice that any sword in storage will degrade by one level for every day that passes. Interesting...

Let's put the sword in storage, and head home. We'll be asked for how many days we would like to sleep. We already fooled the game once. Can we do it again? Try entering `-1`.

```
Your home.
Here you can take a rest.
How many days do you want to rest for? > -1
Sleeping for -1 days
A sword in storage has degraded from 5 to 6.
You woke up well rested.
```

Nice! Our sword has "degraded" to Level 6! Now we grab our sword back from storage, pass time until at least 18:00, and pay the dragon another visit.

```
-------------------------------
Welcome to the dragon's cave
-------------------------------
You see the dragon sleeping next to a pile of bodies.
They look disturbingly fresh.
Carrying your level 6 sword, you walk over to the dragon
You're still dizzy from the time travelling
About half way towards the dragon, the blade starts vibrating
As if by magic, it's pulled out of your hands, and towards the dragon
As it reaches approximately Mach 2 right before impact, you take cover behind a cliff

The impact can only be compared to a small bomb. The entire cave shakes not unlike during an earthquake.
As you look up from you cover, you see the level 6 sword floating in place, just where the dragon used to be.
You walk up to the sword and inspect it closely.

On the blade you can see a faint inscription. You are pretty sure this wasn't here before:

HackTM{g3t_m0re_sl33p_and_dr1nk_m0re_water}
```
Bingo! We've got our flag ;)

### Script
```
#!/usr/bin/env python

from pwn import *

host = '138.68.67.161'
port = 60004

context.log_level = 'CRITICAL'
io = connect(host, port)
print('Working...')
io.sendlineafter('>', '2')
io.sendlineafter('>', '9e9')    # Let's get $9 billion. Cuz I like money.
print('Buying Level 5 sword...')
io.sendlineafter('>', '1')
io.sendlineafter('>', '5')
print('Depositing sword in storage...')
io.sendlineafter('>', '5')
io.sendlineafter('>', 'y')
print('Sleeping until yesterday...')
io.sendlineafter('>', '4')
io.sendlineafter('>', '-1')
print('Grabbing sword...')
io.sendlineafter('>', '5')
io.sendlineafter('>', 'y')
print('Killing time...')    # dragon will kill us if we go during the day
io.sendlineafter('>', '5')
io.sendlineafter('>', 'n')
print('Goiing to see about a dragon...')
io.sendlineafter('>', '3')
resp = io.recvline().decode()
if 'awake' in resp:
    print('He was still awake. DEAD.')
    io.close()
    quit()
print('Slaying a dragon...')
resp = io.recvuntil('}').decode()
io.close()
flag = resp[resp.index('HackTM'):]
print('\n{}'.format(flag))
```
