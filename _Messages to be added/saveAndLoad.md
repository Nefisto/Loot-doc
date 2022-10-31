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