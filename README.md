# CakePHP REST API documentation

## Step 1: Intsall CakePHP using Composer

```bash
composer create-project --prefer-dist cakephp/app cake_rest_api
```
## Step 2: Enable REST API 

To enable REST API we need to add a few lines to [setup resource routes](https://book.cakephp.org/3.0/en/development/routing.html#resource-routes) in your config/routes.php file.

```php
// In config/routes.php...

Router::scope('/', function ($routes) {
    // Prior to 3.5.0 use `extensions()`
    $routes->setExtensions(['json']);
    $routes->resources('Recipes');
});
```

-  where first specifies the desired result format we want. i.e. JSON. 
- Next we mapped REST request to **RecipesController** actions. Now we can move on to creating the logic in our controller actions.

```php
// src/Controller/RecipesController.php
class RecipesController extends AppController
{
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('RequestHandler');
    }

    public function index()
    {
        $recipes = $this->Recipes->find('all');
        $this->set([
            'recipes' => $recipes,
            '_serialize' => ['recipes']
        ]);
    }

    public function view($id)
    {
        $recipe = $this->Recipes->get($id);
        $this->set([
            'recipe' => $recipe,
            '_serialize' => ['recipe']
        ]);
    }

    public function add()
    {
        $recipe = $this->Recipes->newEntity($this->request->getData());
        if ($this->Recipes->save($recipe)) {
            $message = 'Saved';
        } else {
            $message = 'Error';
        }
        $this->set([
            'message' => $message,
            'recipe' => $recipe,
            '_serialize' => ['message', 'recipe']
        ]);
    }

    public function edit($id)
    {
        $recipe = $this->Recipes->get($id);
        if ($this->request->is(['post', 'put'])) {
            $recipe = $this->Recipes->patchEntity($recipe, $this->request->getData());
            if ($this->Recipes->save($recipe)) {
                $message = 'Saved';
            } else {
                $message = 'Error';
            }
        }
        $this->set([
            'message' => $message,
            '_serialize' => ['message']
        ]);
    }

    public function delete($id)
    {
        $recipe = $this->Recipes->get($id);
        $message = 'Deleted';
        if (!$this->Recipes->delete($recipe)) {
            $message = 'Error';
        }
        $this->set([
            'message' => $message,
            '_serialize' => ['message']
        ]);
    }
}
```
In RESTful controllers often we use parsed extensions to serve up different views based on different kinds of requests (ex. JSON, XML, TEXT, HMTL). Since we are dealing with REST API, as a result we need send output either JSON or XML format using CakePHP’s built-in [JSON and XML views](https://book.cakephp.org/3.0/en/views/json-and-xml-views.html).

**Note: There are two ways you can generate data views. The first is by using the _serialize key, and the second is by creating normal template files.**

By using the built in XmlView we can define a _serialize view variable. This special view variable is used to define which view variables XmlView should serialize into XML.

If we wanted to modify the data before it is converted into XML we should not define the _serialize view variable in controller, and instead use template files. We place the REST views for our RecipesController inside src/Template/Recipes/xml. We can also use the Xml for quick-and-easy XML output in those views. Here’s what our index view might look like:
```php
// src/Template/Recipes/xml/index.ctp
// Do some formatting and manipulation on
// the $recipes array.
$xml = Xml::fromArray(['response' => $recipes]);
echo $xml->asXML();
```

