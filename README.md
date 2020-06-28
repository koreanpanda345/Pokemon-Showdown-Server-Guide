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
ghosunny : { // make sure this is lowercase and there is no whitespaces.
	num: -5002, // Pokedex number. its better to start from -5002, and go down from there. The last CAP mon is -5001 that smogon made. 
	Name: “Ghosunny”,// This is the Pokemon's name.
	Types: [“Normal”, “Ghost”], // This is the typing. If you are doing mono type, then do the following
//	Types: ["Normal"], // this is for monotyping.
	gender: “F”, // This is for gender specific pokemon. if your pokemon is either Male or Female, then do the one below
//	genderRatio: {M: 0.5, F: 0.5}, // This is Gender ratio. The ratios are in decemial numbers, from 0 to 1.
	baseStats: { hp: 115, atk: 115, def: 125, spa: 75, spd: 100, spe: 110}, // This is the Base Stats for the pokemon.
	abilities: {0: “Prankster”, 1: “Cursed Body”, H: “Cloak of Nightmares”}, // This is the Pokemon's ability. the H means Hidden. 0, and 1 is the ability slot.
	heightm: 10, // This is height in meters. This is required, i tried doing it with out it, and it complained a lot.
	weightkg: 30, // This is weight in kilograms. This is required, I tried doing it with out it, and it complained a lot.
	color: “Purple”, // This is the Pokemon's Majority color.
	eggGroups: [“Undiscovered”], // This is the Pokemons Egg groups. If you need to add another group, just do the following.
//	eggGroups: ["Monstor", "Bug"], // This is for Pokemon that are in 2 egg groups.
},
```
**Note:** when you are putting in the pokemon name in as the object, make sure there is no whitespaces, and all characters are lowercase.
all of this is required.
Now you must be wondering, "Cloak of Nightmares" isn't an ability. and you would be right, thats because it is a custom ability.
To make this specific ability we would have to do something like this in the server/data/abilities.ts file.
```ts
    cloakofnightmares: { // Make sure this is lowercase, and there is no whitespaces.
        shortDesc: "If a pokemon makes Contact with Ghosunny, Their Attacks is dropped by 1 stage.", // this is the short description of the ability.
        desc: "When a pokemon makes contact with Ghosunny, they see a nightmare, causing them to shake in fear.", // This is the description of the ability.
        onDamagingHit(damage, target, source, move) { // onDamageHit() is called on when the target is hit by a Damaging move.
            if(move.flags['contact']) { // Checks if the move flags has contact.
	    //If so, then
                this.add('-ability', target, 'Cloak of Nightmares'); // Write to the battle logs
                this.boost({atk: -1}, source, target, null, true); // Lower the source's attack by 1 stage.
            }
        },
        name: "Cloak of Nightmares", // The Ability's Name
        rating: 10, // The Ability's Rating.
        num: -100 // The Ability number. It is best to have - next to the number so there is no conflicts.
    },
```

onDamagingHit() is called on when the target(the user) is hit by a damaging move from the source(the attacking pokemon). In this ability case, it will lower the attacking pokemon's attack by 1 stage if it makes contact with Ghosunny.
   
   Still with me? Good.
   
   Now lets do a custom move.
   This one is called Peek-a-Boo. Its an interesting move that I came up with
```ts
   peekaboo: { // Make sure this is lowercase and there is no whitespace
        name: "Peek-a-Boo", // the move's name
	// This is the Short description of the move.
        shortDesc: "If the target is asleep, This move will decrease all of the target’s stats by 1 stage. This will wake up the target. Fails if the target is not asleep.",
        // This is the description of the move.
	desc: "When the target is sleeping, the user manifests itself into the target’s dream, and terrorizes the target inside their mind. Causing the target to wake up feeling weak.",
        effect: { // This is an optional property. But this is used when your move has effects. effect and secondary are not the same thing in this context.
            onHit(target, source, move) { // onHit() is called on when the target is hit by the source. in this context, when the user hits the Target.
                if(target.status !== 'slp') return; // Checks if the target's status is asleep. if not, then return. which in this context means the move will fail.
                target.cureStatus(); // This will cure the status of the target. in this context it removes the asleep status.
                this.boost({atk: -1, def: -1, spa: -1, spd: -1, spe: -1}, target, source, move); // this.boost will lower the target's status 1 stage.
            },
        },
//	secondary: null, //This is for moves that have a secondary effect, such as burn, para, etc.
	category: 'Status', // The move category. in this context it is a Status move.
	target: 'normal', // The move's target type. in this context it is normal, meaning it will hit only one of the pokemon.
        pp: 30,// Power Points. the amount of times the move can be used.
        priority: 0,// Priority of the move. in this context it doesn't have priority so it is 0.
        accuracy: true,// the move's accuracy. If the vaule is set to true, like in this context, that means it will never miss. if your move can miss, then you would put in a number.
        basePower: 0, // the move's base power. In this context, since it is a status move, it doesn't have a base power.
        flags: {status: 1}, // the move's flags. this is what kind of move is it. does it make contact, can it be blocked by protect, etc. all of the flags values are numbers, and should be 1, which means true in this context. if the flag doesn't apply to the move, then don't add it.
        type: "Ghost", // The move's typing.
        contestType: "Cool", // The Contest type.
    },
```
   onHit() is called on when the source hits the target. This can be either a status move, or a damaging move.
   
   now we need to add the Ghosunny's learnsets.
   I am just going to add the Peek-a-Boo move, and maybe hypnosis to test the move out.
   
   to do this, we need to go to the server/data/learnsets.ts file, and do the following:
```
   ghosunny: { // Make sure this is lowercase and there is no whitespace. this also need to be spelt just like how it is in the pokedex.ts file.
      learnsets: { // This is creating the learnsets object for the pokemon.
        peekaboo: ["8L1"], // This is one of the moves that it can learn.
        hypnosis: ["8L1"], // Another move.
      },
   },
```
   The 8L1 just means Gen 8, Level up, at Level 1.
   
   now we can do build the files. if you are windows, you already know how, but if you are not on windows, just simply open a console, and go to the client/build-tools, and do `node <build tool name>` with out the <>.
   Done?
   Neat, lets run the client, and the server, open two consoles in the client directory. in one of them type in `npx http-server` and in the other do `cd data/pokemon-showdown && node pokemon-showdown`. leave those open.
   
   now go to any web browser, and type in `localhost:8080/testclient.html?~~localhost:8000`
   and test see if the pokemon you made is there.
   
   This is all for now, I am still playing with it, and trying to find new stuff. I will make updates here after i find something useful
