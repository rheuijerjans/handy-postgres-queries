# Roles

Get the roles that a user is member of. Note that the 'user' is also a role.
```sql
WITH RECURSIVE cte AS (
   SELECT oid FROM pg_roles WHERE rolname = 'maxwell' -- change name
   UNION ALL
   SELECT m.roleid
   FROM   cte
   JOIN   pg_auth_members m ON m.member = cte.oid
   )
SELECT oid, oid::regrole::text AS rolename FROM cte;  -- oid & name
```

[source: Stackoverflow](https://dba.stackexchange.com/questions/56096/how-to-get-all-roles-that-a-user-is-a-member-of-including-inherited-roles)
