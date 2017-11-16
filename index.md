# *Eventseries*
---
## *Wat?*
---
Een reeks van *events*.
---
1. Een tijdsaanduiding: *start* en een *eind*.
2. Punt *geometerie* en/of gekoppeld aan een *asset*.
3. Een *waarde* (tekst int of float).
4. Overige *metadata* (waaronder de duur).
---
## *Wat voor data?*
---
# Regulier
---
- Storingen op objecten
- Afwijkingen van waterstanden tov streefpeilen
- twitter melding 
- aardbevingen
---
# Koppeling aan assets
---
- hoogwater moment tpv asset
- overstorten & storingen op assets 
---
## Twee typen aanlevering
---
1. Als *afgeleide* op onze tijdgerelateerde data (bijv. alarmen). 
2. Als melding en of *handmatig* aangebracht.
---
*[voorbeeld](https://demo.lizard.net/favourites/270fe2b1-04a6-425a-9bf2-a4286cd6e4e9)*
---
## *Problemen.*
---
*Joep:*
> Factor 10 meer events.

*Lex:*
> Potentieel miljoenen.
---
*Roel:*
- Logica in de client ipv backend. 
- Performance bij > 5000 events.
---
# Performance?
---
Geindexeerde queries nodig op:
1. een tijdsaanduiding: start en een eind.
2. een 2D punt geometerie dus een x en y
---
# *Voorgestelde oplossing*
![4D](http://i.imgur.com/iLMp4MV.gif)
---
# aggregaties
---
- Events hebben 3 dimensies: x, y en tijd, en vormen een lijn in de tijdsdimensie (de duur)
- Events worden altijd opgevraagd binnen de context van een Eventseries.
---
- ruimtelijk afhankelijk van het zoomniveau hierarchisch clusteren
- in de tijd is aggregatie relevant op afgeronde tijdsstappen: 5 minuten, uur, dag, week?, maand, jaar
---
# Enter *Materialized views*

> "A materialized view is a snapshot of a query saved into a table."
---
Werkt niet out of the box:
- Alleen snapshot, refresht niet uit zichzelf.
---
? oplossing ? :
```
REFRESH MATERIALIZED VIEW CONCURRENTLY
```
---
Nope.
---
- Eager rows updaten?
- Lazy rows updaten?
Zie [deze blogpost](https://hashrocket.com/blog/posts/materialized-view-strategies-using-postgresql)
---
```SQL
create function lazy.refresh_account_balance(_name varchar)
  returns lazy.account_balances_mat
  security definer
  language sql
as $$
  with t as (
    select
      coalesce(
        sum(amount) filter (where post_time <= current_timestamp),
        0
      ) as balance,
      coalesce(
        min(post_time) filter (where current_timestamp < post_time),
        'Infinity'
      ) as expiration_time
    from transactions
    where name=_name
  )
  update lazy.account_balances_mat
  set balance = t.balance,
    expiration_time = t.expiration_time
  from t
  where name=_name
  returning account_balances_mat.*;
$$;
```
---
# queries
---
- *Postgis 3 (of 4D)* waarbij 1 wordt ingenomen door de tijd
- *R-Tree* index
---
![RTree 3D](https://upload.wikimedia.org/wikipedia/commons/5/57/RTree-Visualization-3D.svg)
---
![RTree 2D](https://upload.wikimedia.org/wikipedia/commons/6/6f/R-tree.svg)
---
# alternatieve query
---
*Postgis gecombineerd met een 2 tijdsindexen (start en eind)*
- als R-Tree niet zo goed werkt
- invloed temporele dimensie (te groot of te klein)
---
# *Ja maar...*
---
- Geen out-of-the-box oplossing voor materialized views die eager of lazy partial updaten.
- Moet Postgresql functies schrijven.
- niet eindeloos schaalbaar (geen big data).
---
# *vragen Lex*
---
- koppeling met assets?
- assets vs geometrie = dat werkbaar in één?
- Moeten we timeseries- en rasteralarms gaan opslaan als events?
- Annotations al event?
- KPI-dashboard: Welke aggregatie hebben we nodig om PI's performant te maken?
- Attribuut category relevant of niet?
---
-
![Einde](http://cobra.canvas.be/polopoly_fs/1.878568!image/887936073.jpg_gen/derivatives/landscape670/887936073.jpg)
-
