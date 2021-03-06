Sequelize fixtures
==========================================

This is a simple lib to load data to database using sequelize.  
It is intended for easily setting up test data.  
Yaml and json formats are supported. Includes a grunt task.  
Duplicate records are not insertd.
API returns bluebird promises, but callbacks can also be used as the last argument.

### Install
    
    npm install sequelize-fixtures

### Test
    
    npm test

### Usage

```javascript
    var sequelize_fixtures = require('sequelize-fixtures'),
        models = {
            Foo: require('./models/Foo')
        };

    //from file
    sequelize_fixtures.loadFile('fixtures/test_data.json', models).then(function(){
        doStuffAfterLoad();
    });

    //can use glob syntax to select multiple files
    sequelize_fixtures.loadFile('fixtures/*.json', models).then(function(){
        doStuffAfterLoad();
    });

    //array of files
    sequelize_fixtures.loadFiles(['fixtures/users.json', 'fixtures/data*.json'], models).then(function(){
        doStuffAfterLoad();
    });

    //specify file encoding (default utf8)
    sequelize_fixtures.loadFile('fixtures/*.json', models, { encoding: 'windows-1257'}).then(function(){
        doStuffAfterLoad();
    });
    
    //apply transform for each model being loaded
    sequelize_fixtures.loadFile('fixtures/*.json', models, {
        transformFixtureDataFn: function (data) {
          if(data.createdAt 
           && data.createdAt < 0) { 
            data.createdAt = new Date((new Date()).getTime() + parseFloat(data.createdAt) * 1000 * 60);
          }
          return data;
        }
    }).then(function() {
        doStuffAfterLoad();
    });

    //from array
    var fixtures = [
        {
            model: 'Foo',
            data: {
                propA: 'bar',
                propB: 1
            }
        },
        {
            model: 'Foo',
            data: {
                propA: 'baz',
                propB: 3
            }
        }
    ];
    sequelize_fixtures.loadFixtures(fixtures, models).then(function(){
        doStuffAfterLoad();
    });
```

### File formats

#### json

```json
    [
        {
            "model": "Foo",
            "data": {
                "propA": "bar",
                "propB": 1
            }
        },
        {
            "model": "Foo",
            "data": {
                "propA": "baz",
                "propB": 3
            }
        }
    ]
```

#### yaml

```yaml
    fixtures:
        - model: Foo
          data:
            propA: bar
            propB: 1
        - model: Foo
          data:
            propA: baz
            propB: 3
```


#### javascript

```javascript
    module.exports = [
        {
            "model": "Foo",
            "data": {
                "propA": "bar",
                "propB": 1
            }
        },
        {
            "model": "Foo",
            "data": {
                "propA": "baz",
                "propB": 3
            }
        }
    ];
```


### Associations 

You can specify associations by providing related object id or a where clause to select associated object with. Make sure associated objects are described before associations!

#### belongsTo

Assuming `Car.belongsTo(Owner)`:


```json
[
    {
        "model": "Owner",
        "data": {
            "id": 11,
            "name": "John Doe",
            "city": "Vilnius"
        }
    },
    {
        "model": "Car",
        "data": {
            "id": 203,
            "make": "Ford",
            "owner": 11
        }
    }
]
```

OR 

```json
[
    {
        "model": "Owner",
        "data": {
            "name": "John Doe",
            "city": "Vilnius"
        }
    },
    {
        "model": "Car",
        "data": {
            "make": "Ford",
            "owner": { //make sure it's unique across all owners
                "name": "John Doe" 
            }
        }
    }
]
```

#### hasMany, belongsToMany

Assuming 

```javascript
Project.hasMany(Person);
Person.hasMany(Project);
```

or

```javascript
Project.belongsToMany(Person);
Person.belongsToMany(Project);
```

```json
[
    {
        "model":"Person",
        "data":{
            "id":122,
            "name": "Jack",
            "role": "Developer"
        }
    },
    {
        "model":"Person",
        "data":{
            "id": 123,
            "name": "John",
            "role": "Analyst"
        }
    },
    {
        "model":"Project",
        "data": {
            "id": 20,
            "name": "The Great Project",
            "peopleprojects": [122, 123]
        }
    }

]
```

OR


```json
[
    {
        "model":"Person",
        "data":{
            "name": "Jack",
            "role": "Developer"
        }
    },
    {
        "model":"Person",
        "data":{
            "name": "John",
            "role": "Analyst"
        }
    },
    {
        "model":"Project",
        "data": {
            "name": "The Great Project",
            "peopleprojects": [
                {                        
                    "name": "Jack"
                },
                {
                    "name": "John"
                }
            ]
        }
    }

]
```

If using Sequelize 3.0.0 or later, you can define the associated resources by their name instead of the relation name, like this:

```json
[
    {
        "model":"Person",
        "data":{
            "name": "Jack",
            "role": "Developer"
        }
    },
    {
        "model":"Person",
        "data":{
            "name": "John",
            "role": "Analyst"
        }
    },
    {
        "model":"Project",
        "data": {
            "name": "The Great Project",
            "people": [
                {                        
                    "name": "Jack"
                },
                {
                    "name": "John"
                }
            ]
        }
    }

]
```

#### Build options, save optons

For each model you can provide build options that are passed to Model.build() and save options that are passed to instance.save(), example:

```json
{
    "model": "Article",
    "buildOptions": { 
        "raw": true, 
        "isNewRecord": true
    },
    "saveOptions": { 
        "fields": ["title", "body"] 
    },
    "data": {
        "title": "Any title",
        "slug": "My Invalid Slug"
    }
}

```

#### Detect duplicates based on select fields

In case you want to detect duplicates based on specific field or fields rather than all fields (for example, don't include entities with the same id, even if other fields don't match), you can speficy these fields with a 'keys' property.

```json
{
    "model": "Person",
    "keys": ["email"],
    "data": {
        "name": "John",
        "email": "example@example.com"
    }
},
{
    "model": "Person",
    "keys": ["email"],
    "data": {
        "name": "Patrick",
        "email": "example@example.com"
    }
}

```
In this example only John will be loaded


# grunt task

Gruntfile.js:

```javascript
    grunt.initConfig({
        fixtures: {
            import_test_data: {
                src: ['fixtures/data1.json', 'fixtures/models*.json'],
                models: function () {  //returns mapping model name: model
                    return require('../models') 
                },
                options: { //specify encoding, optiona. default utf-8
                    encoding: 'windows-1257'
                }
            }
        }

    });

    grunt.loadNpmTasks('sequelize-fixtures');
```