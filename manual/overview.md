# Overview


​	This overview was designed to provide you with the necessary information to fully understand how this plugin work, also create a terminology to help you to understand what something does just by seeing it's name. You can also find a small guide to demo inside the Demo folder that will provide you with an explanation about what each sample tries to demonstrate, it's a good way to understand it by looking directly at the commented code, if you are getting trouble understanding some specific API, just hit me up in discord and I'll try my best to help.

---

## Core concepts

Let's walk thought the core classes:

- **Drop table:** This is the drop container and control the calculation used thought the drops inside it (see below for more information about percentage calculation kind), also it's from here that you will control the *drops* properties, aside from other additional, for more specific cases, settings. It's inherit from scriptable object so you can create it the right-click context - **right-click -> Create -> Drops -> New Table** -. This has not only your drops but also the callbacks, modifier, and filters (we will talk more about these in the specific section), and it's from where you will request drops.
- **Drops:** A layer representing your drop settings, can only be accessed thought the drop table (on the inspector), controlling things like your entry, percentage to drop, amount variation, and other additional settings.

Those classes above you, probably, will change thought the inspector to setup your scenario, below we have classes that represent the answer of some requested operation and only exist in the code side.

- **Bag:** This is a container of *Loot*, when you request a drop from your *Drop table* you will get a Bag as result.
- **Loot:** This is a limited version of the drop that contains only the entry and the collapsed dropped amount (if your *drop* can vary between 3~7 units when randomized a *loot* will have a single value between the selected range), when you iterate over *Bag* you'll get *Loot*.

---

## Additional concepts

Let's take a look at some additional important concepts that reside inside **Loot**:

### Percentage calculation

In the drop table inspector we can see an enum field called **percentage calculation**. This field will change the way that drop percentage/chance is calculated 

![sample](images/sample_percentageCalculation.png)

* **Simple:** Each drop has its percentage individually, this means that when we ask for a drop, the table will try to drop each entry individually. 
  **i.e.** suppose that we have two drops A and B, and each one has 50% to be dropped, when we ask for a drop, our result bag can have A, A and B, B or nothing (empty bag), also as the entries are independent changing the percentage of one entry will not interfere in another percentage.

- **Weighted:** The percentage of each drop depends on the sum of the weight of each item, and the sum of all percentages should always be 100%, this also means that when we ask for a drop the table will give to us only one drop from the list. 
  **i.e.** continuing with our A and B example, if each one weights 1, this means that each has 50% to be dropped, our result bag possibilities are A or B, also if we change our B to weight 2 this will make our A change from 50% to 33% and our B change from 50% to 66%, because the sum of weighs is 3, A percentage is equal to their weight (1) divided by the table weight (3), so A = 1/3 = 33% and B = 2/3 = 66%.

### Extension drop

​	This feature is created to allow you to have hierarchy drops mixing multiple drop tables and treat all then as a unique table. Take the following example in count: in your game every <u>wolf</u> drops a *fur*, also every <u>baby wolf</u> drops a *teeth* then you have a <u>Blue Baby wolf</u> enemy that, as the name suggests, drops things from <u>wolf table</u>,  <u>baby wolf table</u> and also some custom items specific from this kind of wolf. You can toggle this feature into the drop on inspector:

![sample](images/sample_extensionDrop.png)

**Important:** Note that we can enable it on *_Common wolf drop* entry and can't on *Teeth* entry. This option will be enabled only when the entry is another **Drop Table** AND the [percentage calculation](./overview.md#percentage-calculation) between two table are equal.

When this option is enabled the entry will not be treated as a drop (so you will not get it when iterating over this table, instead you will get the union between two drop tables), and in the case that your table is a weighted one, the weight from inner tables will interfere in outer tables, as we can see in the following gif.

![](images\sample_innerDropChangeWeight.gif)

### Modifier

This represents temporary changes that we apply to our drops in runtime that allow us to change drops properties based on custom conditions. For example, increase the drop percentage of consumables based on my sugar eat skill level. Modifiers can be applied in a global or local context, the order runs as the following:

​	**Global modify** => **Global modified** => **Local modify** => **Local modified** => (**Filter part**)
