---
uid: manual.md

---

# Loot - A drop table solution

First of all, thanks for buying and supporting my work, I'm trying my best to keep updating this asset whenever I can, I hope that it helps you to bring your dream game to life, and please if you have some suggestions, critique or question does not hesitate to enter in contact through email or discord.

**Developer contact**

* Email: joao.gavron@gmail.com
* Discord: Nefisto#3403

doc version: 1.2

---

## Architecture

I tried to make the plugin as accessible as possible so here we will take a look about how the asset is organized

This user guide was designed to provide you with the necessary information to fully understand how this plugin work. You can also find a small guide to demo inside the Demo folder that will provide you with an explanation about what each sample tries to demonstrate, it's a good way to understand it by looking directly at the commented code. ATM I'm working at a web scripting API reference, so if you are getting trouble understanding some specific API, just hit me up in discord and I'll try my best to help you.

---

# Quick start

Steps to use the Loot plugin:

1. Import Loot plugin from package manager

2. **Right-click -> Create -> Loot -> New table**

3. Configure your table

4. 1. Chose a **kind** based on what you're planning to do
   2. [Optional] Set a custom logic (let it to none and the default logic will be used)
   3. Add your drops in **Drops** list

5. Create a reference to your table in your desired script with public DropTable myTable 

6. Call the desired API using your reference var bag = myTable.Drop(), all of them will return a bag

7. Your bag contains your drop, so you can iterate over your bag to do what you want with your loot

OBS: Drop table implement IEnumerable<Drop> and run these events inside it, so if you try to do some LINQ inside some event you will end with a stack overflow error, this is why every event give to you a list of modified drops.

---

## System components overview

Let's understand how the system was been architectured, we have five core classes to cover:

- **Drop table:** This is our main container that will contain our drops, percentages, and amounts, together with extra information. It's a scriptable object that can be created on right-click context - **right-click -> Create -> Drops -> New Table** - and control almost every aspect of our drops. This has not only our drops but also our callbacks, modifier, and filters (we will talk more about this in the specific section), and it's from where you will request drops.
- **Drops:** This represents your drop by itself, controlling things like your entry, percentage (in the simple and weighted scenario), amount variation, and some other settings. This exists only inside the context of a *Drop table*.
- **Bag:** When you request a drop from your *Drop table* you will get a Bag as result, and this will contain your randomized drops. 
- **Loot:** When you iterate over Bag you'll get Loots, It is a limited vision of Drop that contains only the entry and a collapsed amount (by collapsed I mean, if your drop entry-amount can vary between 2 and 5, a loot will have a number between 2 and 5).
- **Logic:** This is the logic that will run in the background when you call for a drop using the *Drop Table*, you can inject this on each table or can use the default one that provides one version for each weighted and simple table.

**So the flow was planned to be something like this:** You will create a *Drop Table* and fill it with your *Drops*, then you can ask for a drop from your table that will run the default or custom *Logic* and you will receive a *Bag* as result, then you can iterate over your bag getting the *Loot* version of drops and populating what you were originally working on.

Together with classes I also want to talk about three concepts that reside in the plugin:

- **Drop table kind:** In drop table inspector we can see an enum field called drop table kind. This field will change the way that drop percentage/chance is calculated 

* - **Simple:** Each drop has its percentage individually, this means that when we ask for a drop, the table will try to drop each drop individually. For example, suppose that we have two drops A and B, and each one has 50% to be dropped, when we ask for a drop, our result bag can have A, A and B, B or nothing (empty bag), also changing percentage of one item will not interfere in another item percentage.
  - **Weighted:** The percentage of each drop depends on the sum of the weight of each item, and the sum of all percentages should always be 100%, this also means that when we ask for a drop the table will give to us only one drop from the list. For example, continuing with our A and B example, if each one weights 1, this means that each has 50% to be dropped, our result bag possibilities are A or B, also if we change our B to weight 2 this will make our A change from 50% to 33% and our B change from 50% to 66%.

* **Modifier:** This represents temporary changes that we apply to our drops in runtime that allow us to change drops properties based on custom conditions. For example, increase the drop percentage of consumables based on my sugar eat skill level. Modifiers can be applied in a global or local context, the order runs as the following:

  ​	**Global modify** => **Global modified** => **Local modify** => **Local modified** => (**Filter part**)

  

  OBS: xxxModify happen for every item while xxxModified happen one time, so if your modify condition depends on some value that needs to be counted every time, use the modified event version

  * **Global modify:** 
    * It's raised one time for every drop.
    * It's shared between all tables. 
    * It's an Action<ModifyEventArgs>
  * **Global modified:** 
    * It's raised one time, after all, modify actions finished being applied.
    * It's shared between all tables. 
    * It's an Action<ModifiedEventArgs>
  * **Local modify:** 
    * It's raised one time for every drop.
    * It's individual to each table. 
    * It's an Action<ModifyEventArgs>
  * **Local modified:** 
    * It's raised one time, after all, modify finished being applied.
    * It's individual to each table. 
    * It's an Action<ModifiedEventArgs>.

* **Filter drops:** This will allow us to temporarily filter our drops. For example, in N attempt make sure that users receive at least one of this specific item type in their drop. Filters happen after the modifiers was applied, this means that your condition is based on a modified version of your table.

  * **Filter:** This will remove the drop from the collection if ANY listener returns false.
    * It's raised one time for every drop.
    * It's individual to each table. 
    * It's an Predicate<FilterEventArgs>. 
  * **Filtered:** It's the last thing to be applied, so the list of drops at this point is already modified and filtered
    * It's raised one time. 
    * It's individual to each table.
    * It's an Action<FilteredEventArgs>.


---

# Manual

## Save/Load:

> We have an example scene with a simple and complex case in **Samples -> SaveLoad demo**, I tried to comment on every important part, but if something sounds strange to you, do not hesitate to enter in contact using discord provided at the start of this document. 
>
> For **simple** case look at: **Samples -> SaveLoad demo -> Scripts -> SaveLoad.cs** methods **Simplest Save** and **Simplest Load**
> For **complex** case look at: **Samples -> SaveLoad demo -> Scripts -> SaveLoad.cs** methods **Complex Save** and **Complex Load**

​	Sometimes when using runtime clone tables is necessary to keep their modified values between runs, for example, if the player can get random cards from a box and the got cards were removed from the box we need to keep the current state of the box between runs, with this kind of scenario in mind we made some API's to simplify the process.

​	We'll box and unbox our necessary information inside a custom class made to be serialized, the **DropTableSaveData**, this class is what you'll get from saving and what you'll use to load your tables. We've made this feature thinking in two cases:

1. **Simple case:** Your changes made in the runtime table is all about their values (percentage, weight, drop amount variation, etc...) and/or removing entries
2. **Complex case:**  You've added new entries that do not exist in the original table and/or change entry values (read change the entry reference in runtime) to a custom one that also does not exist in the original table

In a nutshell, if you can restore your runtime table state going from the original table, you'll probably need just the simple case and otherwise the complex case.

**IMPORTANT NOTE 1:** Keep in mind that we will load values to an existing runtime table, so you are responsible to recreate a clone table from your original table.

**IMPORTANT NOTE 2:** Modifiers/filters ARE NOT serialized, so you must get it from the original table and/or re-add any custom modifier/filter added in some kind of setup stage.
