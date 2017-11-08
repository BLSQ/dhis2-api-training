# DHIS2 API

--

## Intro

---

### What

- Officially, a way for other software to get infos out of DHIS2
- Also, a way for human beings to get data they can't get using the UI and make some changes

---

DHIS2 API has tree big advantages:

- It exists
- It is complete
- It works
- It is documented

---

### Why

Answers questions such as:

- What groups is the Entity "CHP Centre de santé" in?
- Give me the list of all entities org unit without GPS coordinates
- Give me the list of entities in the Kwango province with their hierarchy
- Which data sets contains the Data Element "Accouchements assisté"

but also make operations which are costly/complex on the UX side easier, such as manipulating groups (adding/removing entities)

Not talking about getting values for now - that's for another meeting

--

## Basic concepts

---

### DHIS2 Model

This is important to understand relationships

![image](https://yuml.me/d725b413)

Source: http://yuml.me/edit/d725b413

---

### Ids and where to find them:

[Maintenance > Right click > Details > id field](http://dhis2-cd-hlt-prod.herokuapp.com/dhis-web-maintenance/#/list/organisationUnitSection/organisationUnit?_k=8on3m3)

---

### Version

The model evolves a bit with versions, two examples:

- From 2.26, `Org Unit Groups` can be part of multiple `Sets`
- From 2.24, there is a `DataSetElements` between `DataSets` and `DataElements`

---

### First example

A very simple example using DRC and the browser:

    http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits

- Route: `/api/organisationUnits`
- Every API call starts with `/api`

Going to the root gives you a list of what is possible:

    http://dhis2-cd-hlt-prod.herokuapp.com/api

This will not work if not logged - log yourself first using the browser

--

## Better tool: Postman

---

### About

<img src="https://www.getpostman.com/img/v2/logo-glyph.png" alt="Postman" style="width: 200px;"/>

[Chrome extension](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)

- Better output
- Can save queries
- Can pass parameters

---

### First example in postman

- Same url
- Need to add your user/password (can be saved!)

    http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits

---

### Getting a single unit

    http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits/pL5A7C1at1M

A lot of info there (maybe too much)

--

## Listing things

---

### Paging

Info in the request:

     "pager": {
        "page": 1,
        "pageCount": 563,
        "total": 28102,
        "pageSize": 50,
        "nextPage": "http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?page=2"
    }

---

Change the page size:

- [/api/organisationUnits?pageSize=500](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?pageSize=500)
- [/api/organisationUnits?pageSize=30000](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?pageSize=30000)

---

### Output format

Default is JSON, but can be changed:

[/api/organisationUnits.csv](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits.csv)

CSV is very nice (excel like) but more limited (why?), we'll see what we can do about it

---

### Fields simple

List by default show only id and names - but we can change that:

[/api/organisationUnits?fields=id,name,level](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,level)

Other good example: coordinates

---

### Fields advanced

Can use relationships such as "parent" or "organisationUnitGroups":

[/api/organisationUnits?fields=id,name,level,parent,organisationUnitGroups](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,level,parent,organisationUnitGroups)

---

Using hierarchy `parent[name]`:

[/api/organisationUnits?fields=id,name,level,parent[name,parent[name]]](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,level,parent[name,parent[name]])

---

### Filtering

We don't want all org units - only the DPS - how?

[/api/organisationUnits?fields=id,name,level&filter=level:eq:2](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,level&filter=level:eq:2)

---

### Operators

General structure:

    field:operator:value

- eq (null)
- neq (null)
- ilike
- gt
- lt
- ...

[Doc](https://docs.dhis2.org/2.24/en/developer/html/dhis2_developer_manual_full.html#d7652e1009)

---

### Filtering on names

Org Units containing with Moa:

[/api/organisationUnits?fields=id,name,level&filter=name:ilike:Moa](mhttp://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,level&filter=name:ilike:Moa)

`eq` is bad on names - use `ilike` - we could limit this to entities, not zones using `level`

---

### Filtering on anything

Anything can be used in a filter - even parents for example:

[/api/organisationUnits?fields=id,name,level&filter=parent.parent.name:ilike:Equateur](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,level&filter=parent.parent.name:ilike:Equateur)

---

or being part of a specific group:

[/api/organisationUnits?filter=organisationUnitGroups.id:eq:pwzfZKozJCW](http://dhis2-cd-hlt-prod.herokuapp.com/api/organisationUnits?fields=id,name,organisationUnitGroups&filter=organisationUnitGroups.id:eq:pwzfZKozJCW)

There is an easier way to do this, but it's still useful (why?)

--

## Other meta

---

### Data Elements

It's just the same...

[/api/dataElements?fields=id,name,aggregationType](http://dhis2-cd-hlt-prod.herokuapp.com/api/dataElements?fields=id,name,aggregationType)

Same for `DataSet`, `OrganisationUnitGroup`, etc

---

## Back to CSV

JSON to CSV converter: https://json-csv.com/ 

--

## Modifying things

---

### Warning

Be careful from here.

---

### Relation

A common case is adding and removing elements to groups (org unit to org unit groups, data element to data element groups, etc)

    POST /api/organisationUnitGroup/:groupid/organisationUnits/:unitid

---

So:

    POST /api/organisationUnitGroups/aPwKBQKIlCr/organisationUnits/ugs8aAdEvgc

---

This would add org unit `ugs8aAdEvgc` to group `aPwKBQKIlCr`

    DELETE /api/organisationUnitGroups/aPwKBQKIlCr/organisationUnits/ugs8aAdEvgc

would remove it

---

This is not possible in browser but can be done in POSTMAN

On play: https://play.dhis2.org/demo/api/organisationUnitGroups/CXw2yu5fodb/organisationUnits/EURoFVjowXs

--

## Getting help

---

### At BLSQ

- Channel: #dhis2
- Team: NG, dev team (MVA, SMS)

---

### Documentation

https://docs.dhis2.org 

* Develper guide (use HTML one page for search) 
* Mind the version!

---

### DHIS2 mailing list

https://www.dhis2.org/contact#mailing-lists

--

## Thanks!