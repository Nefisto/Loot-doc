# Quick start

Steps to use the Loot plugin:

1. Import Loot plugin from package manager
2. Create a new table thought: **Right-click -> Create -> Loot -> New table**
3. Configure your table
4. Fill the drop list (here you need to make sure that your *item concept* inherit from scriptable object, because this is the base type used to add drops thought inspector)
5. Create a reference to your table in your desired script
6. Call the desired API using your reference and store the bag result ```var bag = myTable.Drop()``` (all API's returns a bag)
7. Your bag contains your drop, so you can iterate over your bag to do what you want with your result
