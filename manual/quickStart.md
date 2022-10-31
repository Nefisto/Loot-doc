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