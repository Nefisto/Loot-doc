* OBS: xxxModify happen for every item while xxxModified happen one time, so if your modify condition depends on some value that needs to be counted every time, use the modified event version

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



OBS: Drop table implement ```IEnumerable<Drop>``` and run these events inside it, so if you try to do some LINQ inside some event you will end with a stack overflow error, this is why every event give to you a list of modified drops.