# Pokemon-Showdown-Server-Guide
This is more of a guide on how to add fakemons to a custom pokemon showdown server.

Making a pokemon showdown server is some what complicated, and adding a fakemon to it is even harder to do. Well thats why this was made.
This guide will help make it somewhat easier to add your own custom stuff to your server. 

# Disclaimer
I do want to say that it is a good idea to have some kind of basic knowledge in programming. This will make it a lot easier if you have basic knowledge.
Also should mention, that I am going to assume that you already have everything you need. so I will not go over that.
This guide is more specific for the client side and adding fakemons to it. not to setup a server. you can read the README.md file in the server repo to find out how to set it up.

## How do I add a fakemon?
well the first thing to know is that you will need to have the server side folder in a specific spot, which is in the client/data folder. But it is easier to use a console, and type in `npm run build-full`. This will build a server side folder in the data folder for us. You maybe wondering why, the answer is of course the build tools.

### Build-Tools
In the client folder there is a folder called build-tools. this folder is going to be your best friend. This folder has files that will build the data for the client for us. so we don't have to copy and paste files in.
- build-indexes.js : This file is used to make the search-indexes.js file which is a needed to have.
- build-learnsets.js : This file is used to build the learnsets.js and learnsets-g6.js files. this is needed for the move pool for the pokemon.
- build-minidex.js : This file is used to build the pokedex-mini.js and pokedex-minibw.js file. this is needed for the pokemon to be pokemon.
- build-sets.js : this is more for building random-team sets, and 1v1 set data. this is optional.

these files are needed to be executed after you finish making  changes and ready to test.

### If you are on windows, i have a neat tip for you to make it easy.
make a new file in client folder, and call it `build.bat`.
in this file put in :
```
cd data

del mod/
del abilities.js
del aliases.js
del cap-1v1-sets.json
del graphic.js
del items.js
del learnsets.js
del learnsets-g6.js
del pokedex.js
del pokedex-mini.js
del random-team.js
del search-index.js
del rulesets.js
del scripts.js
del statuses.js
del teambuilder-tables.js
del typechart.js

cd ../build-tools && node build-indexes && node build-learnsets && node build-minidex

```
then when you need to build, instead doing each one manual just double click on the `build.bat` file in the file explore and watch the magic happen.

### Ok how do I really had a fakemon in?
Well you need to edit a few files.
``Keep in mind that you will be making any changes in the server folder, not the client folder``
first thing we are going to do is add the pokedex entry. I am going to use Ghosunny, a fakemon That I made.
So to add Ghosunny we need to edit the server/data/pokedex.ts file.
to add a new pokemon you need the following, (I am going add an example.)
```ts
ghosunny : {
	num: -5002,
	Name: “Ghosunny”,
	Types: [“Normal”, “Ghost”],
	gender: “F”,
	baseStats: { hp: 115, atk: 115, def: 125, spa: 75, spd: 100, spe: 110},
	abilities: {0: “Prankster”, 1: “Cursed Body”, H: “Cloak of Nightmares”},
	heightm: 10,
	weightkg: 30,
	color: “Purple”,
	eggGroups: [“Undiscovered”],
},
```
all of this is required.
Now you must be wondering, "Cloak of Nightmares" isn't an ability. and you would be right, thats because it is a custom ability.
To make this specific ability we would have to do something like this in the server/data/abilities.ts file.
```ts
    cloakofnightmares: {
        shortDesc: "If a pokemon makes Contact with Ghosunny, Their Attacks is dropped by 1 stage.",
        desc: "When a pokemon makes contact with Ghosunny, they see a nightmare, causing them to shake in fear.",
        onDamagingHit(damage, target, source, move) {
            if(move.flags['contact']) {
                this.add('-ability', target, 'Cloak of Nightmares');
                this.boost({atk: -1}, source, target, null, true);
            }
        },
        name: "Cloak of Nightmares",
        rating: 10,
        num: -100
    },
```

onDamagingHit() is called on when the target(the user) is hit by a damaging move from the source(the attacking pokemon). In this ability case, it will lower the attacking pokemon's attack by 1 stage if it makes contact with Ghosunny.
   
   Still with me? Good.
   
   Now lets do a custom move.
   This one is called Peek-a-Boo. Its an interesting move that I came up with
```ts
   peekaboo: {
        name: "Peek-a-Boo",
        shortDesc: "If the target is asleep, This move will decrease all of the target’s stats by 1 stage. This will wake up the target. Fails if the target is not asleep.",
        desc: "When the target is sleeping, the user manifests itself into the target’s dream, and terrorizes the target inside their mind. Causing the target to wake up feeling weak.",
        effect: {
            onHit(target, source, move) {
                if(target.status !== 'slp') return;
                target.cureStatus();
                this.boost({atk: -1, def: -1, spa: -1, spd: -1, spe: -1}, target, source, move);
            },
        },
        pp: 30,
        priority: 0,
        accuracy: true,
        basePower: 0,
        flags: {status: 1},
        type: "Ghost",
        contestType: "Cool",
    },
```
   onHit() is called on when the source hits the target. This can be either a status move, or a damaging move.
   
   now we need to add the Ghosunny's learnsets.
   I am just going to add the Peek-a-Boo move, and maybe hypnosis to test the move out.
   
   to do this, we need to go to the server/data/learnsets.ts file, and do the following:
```
   ghosunny: {
      learnsets: {
        peekaboo: ["8L1"],
        hypnosis: ["8L1"],
      },
   },
```
   The 8L1 just means Gen 8, Level up, at Level 1.
   
   now we can do build the files. if you are windows, you already know how, but if you are not on windows, just simply open a console, and go to the client/build-tools, and do `node <build tool name>` with out the <>.
   Done?
   Neat, lets run the client, and the server, open two consoles in the client directory. in one of them type in `npx http-server` and in the other do `cd data/pokemon-showdown && node pokemon-showdown`. leave those open.
   
   now go to any web browser, and type in `localhost:8080/testclient.html?~~localhost:8000`
   and test see if the pokemon you made is there.
   
   This is all for now, I am still playing with it, and trying to find new stuff. I will make updates where after i find something useful
