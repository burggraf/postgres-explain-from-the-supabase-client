# How to Get PostgreSQL Explain / Analyze Statistics from the Supabase Javascript Client

## Please Explain
PostgreSQL has this great tool for analyzing complex queries.  You can use it to see what parts of your query are "costing" you CPU time, and use that information to build the right indexes to make your queries run lightning fast:

Postgres `EXPLAIN` — show the execution plan of a statement

[https://www.postgresql.org/docs/current/sql-explain.html](https://www.postgresql.org/docs/current/sql-explain.html)

This is great when you're writing SQL, but what if you're using the Supabase Javascript client?

```js
const { data, error } = await supabase
  .from('users')
  .select(`
    name,
    teams (
      name
    )
  `)
```

Maybe you tried using `select query from pg_stat_statements` to find the query that your client code is sending to PostgreSQL.  But then you see all sorts of parameters in the query such as `$1`, `$2`, `$3`.  Now how do you analyze this?

There's an easier way -- you can get this data right from your client in debug mode.

## Step One: Turn on `db_plan_enabled`
To be able to debug from the client side, you'll need to enable it first by running these commands:

```sql
alter role authenticator 
set pgrst.db_plan_enabled to true;
notify pgrst, 'reload config';
```

## Step Two: Debug your client queries
Now that you have `db_plan_enabled` turned on you can do this in your client code:

```js
const { data, error } = await supabase
  .from('users')
  .select(`
    name,
    teams (
      name
    )
  `).explain({analyze: true, verbose: true, format: 'json'})
```

This will return valuable information about the query execution plan that you can use to tune your database for better performance.

## Step Three: Turn it off
When you're done debugging, just turn `db_plan_enabled` back off with:

```sql
alter role authenticator 
set pgrst.db_plan_enabled to false;
notify pgrst, 'reload config';
```

## Conclusion
This makes it easier to see exactly what PostgREST (the PostgreSQL REST interface used by the Supabase Javascript library) is doing behind the scenes.  Once you know where the big costs are in your query, you can build the necessary indexes and you should see better performance on those operations.

## Note
You'll need to be on PostgREST 10.1.1 or higher to get this capability.  

To check your version, you can run this:

```sh
curl '<SUPABASE_URL>/rest/v1/' \
-H "apikey: <SUPABASE_ANON_KEY>" \
-H "Authorization: Bearer <SUPABASE_ANON_KEY>" | more
```

You'll see the PostgREST version near the very beginning of the output.
