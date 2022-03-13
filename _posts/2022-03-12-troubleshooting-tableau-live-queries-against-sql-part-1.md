## Troubleshooting performance of tableau live connections to SQL Server

Hello everyone, wish you enjoy reading these findings I've gathered over a couple of years working with this type of scenario

#### Background

Tableau and SQL Server are great tools but they may be troublesome to set working together.
There are use cases when you'd want to use *live* connection as opposed to *extracts*. These include, but not limited to cases when you want to use:
* integrated authentication in the DB,
* minimize storage footprint at the tableau server, 
* or your application requires that live data is retrieved from the DB
and many others.

#### Problem description

An often occuring problem is performance of such connections. You may have just a handful (<< 1mln) of rows in the source tables, but the views would take minutes to load.

