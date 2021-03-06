## Troubleshooting performance of tableau live connections to SQL Server - part 2. DETAILS and ACTION.

👋 Hello everyone, wish you enjoy reading these findings I've gathered over a couple of years working with this type of scenario
This is part 2 where we discuss specific symptoms and troubleshooting steps.

I want to stress that the article is written with love and respect to this wonderful tool - don't misinterpret this as biased critics.

### Not obvious stuff done by Tableau under-the-hood

Here is a list of things that Tableau does to retrieve data for you - and the things that make your SQL Server yield suboptimal performance.

   1. Tableau generates a distinct query for each filter you have on a dashboard,
      Yes, and every time a user presses /refresh/, the DB is asked to retrieve a distinct set of elements to be displayed   
      📝 Remove all unnecessary filters from the dashboard - unless you _really_ need them.
   2. Tableau executes queries sequentially (if there are too many – then it shows in performance),
      Similar stuff as above. The more filters - the more wait time.
   3. WHERE predicates generated by Tableau are often far too complex for the DB to pick the right index and are non-sargable
      As a rule of a thumb, less filters are working better. Don't add a bunch of them just for the rainy day,
      Simple filters are normally better. I.e.:
         `WHERE a=1`
      normally works faster then
         `WHERE a = CASE WHEN someNumberDescription = 'one' THEN 1 ELSE 0 END`   
      📝 Make sure that your filters are expressed as simple as possible and use the unchanged columns from the data source.

   4. Tableau generates bad type conversions (i.e. call everything nvarchar (9000))   
      📝 Ensure that your calculated measures avoid type casts - both implicit and explicit      
      📝 Perform operations and comparisons within the same type - int being more preferred over strings or floats,   
      📝 Avoid concantenation of strings in measures   
      📝 If any of the above is strictly necessary, do them in SQL code.
   
   5. LOD’s.
      LOD's - all of them - normally show up very poor in the SQL code - very detrimental for performance. They can always be replaced with subqueries in SQL that are much more deterministic in performance   
      📝 Use subqueries in SQL vs LOD's    
      
   6. Some formulas yield code like “CASE WHEN 1=0 THEN …. ELSE … END”,\
      If you see them, look for the column they reference and try to drop this column from the dashboard / guess the reason why such structure is added.
   7. Tableau generates ugly joins with subqueries  
      Tableau introduced the 'new' join logic - it generates joins to subqueries, not the individual views. Better place all objects in one container (to phrase it better an place a screenshot to aid understanding).

### Where do I start from?

First step would probably be to assess how bad the SQL query is. In most severe cases, you might want to rebuild your viz from scratch - only adding the necessary things to the screen. But most likely, you'll need just a cleanup. My cleanup sequence goes like this:
1) Drop unnecessary filters from the dashboard
2) Cleanup the WHERE predicates,
3) Remove all subqueries in JOINs
4) Cleanup type casts in the SELECT area,
When you're satisfied with how your queries look like, you can try executing them in SSMS and start with performance optimization on the DB side



