Serializer
==========

The *SerializerServiceProvider* provides a service for serializing objects.

Parameters
----------

None.

Services
--------

* **serializer**: An instance of `Symfony\\Component\\Serializer\\Serializer
  <https://api.symfony.com/master/Symfony/Component/Serializer/Serializer.html>`_.

* **serializer.encoders**: `Symfony\\Component\\Serializer\\Encoder\\JsonEncoder
  <https://api.symfony.com/master/Symfony/Component/Serializer/Encoder/JsonEncoder.html>`_
  and `Symfony\\Component\\Serializer\\Encoder\\XmlEncoder
  <https://api.symfony.com/master/Symfony/Component/Serializer/Encoder/XmlEncoder.html>`_.

* **serializer.normalizers**: `Symfony\\Component\\Serializer\\Normalizer\\CustomNormalizer
  <https://api.symfony.com/master/Symfony/Component/Serializer/Normalizer/CustomNormalizer.html>`_
  and `Symfony\\Component\\Serializer\\Normalizer\\GetSetMethodNormalizer
  <https://api.symfony.com/master/Symfony/Component/Serializer/Normalizer/GetSetMethodNormalizer.html>`_.

Registering
-----------

.. code-block:: php

    $app->register(new Silex\Provider\SerializerServiceProvider());
    
.. note::

    Add the Symfony's `Serializer Component
    <https://symfony.com/doc/current/components/serializer.html>`_ as a
    dependency:

    .. code-block:: bash

        composer require symfony/serializer

Usage
-----

The ``SerializerServiceProvider`` provider provides a ``serializer`` service::

    use Silex\Application;
    use Silex\Provider\SerializerServiceProvider;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    $app = new Application();

    $app->register(new SerializerServiceProvider());

    // only accept content types supported by the serializer via the assert method.
    $app->get("/pages/{id}.{_format}", function (Request $request, $id) use ($app) {
        // assume a page_repository service exists that returns Page objects. The
        // object returned has getters and setters exposing the state.
        $page = $app['page_repository']->find($id);
        $format = $request->getRequestFormat();

        if (!$page instanceof Page) {
            $app->abort("No page found for id: $id");
        }

        return new Response($app['serializer']->serialize($page, $format), 200, array(
            "Content-Type" => $request->getMimeType($format)
        ));
    })->assert("_format", "xml|json")
      ->assert("id", "\d+");

Using a Cache
-------------

To use a cache, register a class implementing ``Doctrine\Common\Cache\Cache``::

    $app->register(new Silex\Provider\SerializerServiceProvider());
    $app['serializer.normalizers'] = function () use ($app) {
        return [new \Symfony\Component\Serializer\Normalizer\CustomNormalizer(),
            new \Symfony\Component\Serializer\Normalizer\GetSetMethodNormalizer(new ClassMetadataFactory(new  AnnotationLoader(new AnnotationReader()), $app['my_custom_cache']))
        ];
    };
