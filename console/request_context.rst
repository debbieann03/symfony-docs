.. index::
   single: Console; Generating URLs

How to Generate URLs from the Console
=====================================

Unfortunately, the command line context does not know about your VirtualHost
or domain name. This means that if you generate absolute URLs within a
console command you'll probably end up with something like ``http://localhost/foo/bar``
which is not very useful.

To fix this, you need to configure the "request context", which is a fancy
way of saying that you need to configure your environment so that it knows
what URL it should use when generating URLs.

There are two ways of configuring the request context: at the application level
and per Command.

Configuring the Request Context Globally
----------------------------------------

To configure the Request Context - which is used by the URL Generator - you can
redefine the parameters it uses as default values to change the default host
(``localhost``) and scheme (``http``). You can also configure the base path (both for
the URL generator and the assets) if Symfony is not running in the root directory.

Note that this does not impact URLs generated via normal web requests, since those
will override the defaults.

.. configuration-block::

    .. code-block:: yaml

        # config/services.yaml
        parameters:
            router.request_context.host: 'example.org'
            router.request_context.scheme: 'https'
            router.request_context.base_url: 'my/path'
            asset.request_context.base_path: '%router.request_context.base_url%'
            asset.request_context.secure: true

    .. code-block:: xml

        <!-- config/services.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

            <parameters>
                <parameter key="router.request_context.host">example.org</parameter>
                <parameter key="router.request_context.scheme">https</parameter>
                <parameter key="router.request_context.base_url">my/path</parameter>
                <parameter key="asset.request_context.base_path">%router.request_context.base_url%</parameter>
                <parameter key="asset.request_context.secure">true</parameter>
            </parameters>

        </container>

    .. code-block:: php

        // config/services.php
        $container->setParameter('router.request_context.host', 'example.org');
        $container->setParameter('router.request_context.scheme', 'https');
        $container->setParameter('router.request_context.base_url', 'my/path');
        $container->setParameter('asset.request_context.base_path', $container->getParameter('router.request_context.base_url'));
        $container->setParameter('asset.request_context.secure', true);

.. versionadded:: 3.4
    The ``asset.request_context.*`` parameters were introduced in Symfony 3.4.

Configuring the Request Context per Command
-------------------------------------------

To change it only in one command you can fetch the Request Context from the
router service and override its settings::

    // src/Command/DemoCommand.php
    use Symfony\Component\Routing\RouterInterface;
    // ...

    class DemoCommand extends ContainerAwareCommand
    {
        private $router;

        public function __construct(RouterInterface $router)
        {
            $this->router = $router;
        }

        protected function execute(InputInterface $input, OutputInterface $output)
        {
            $context = $this->router->getContext();
            $context->setHost('example.com');
            $context->setScheme('https');
            $context->setBaseUrl('my/path');

            $url = $this->router->generate('route-name', array('param-name' => 'param-value'));
            // ...
        }
    }
