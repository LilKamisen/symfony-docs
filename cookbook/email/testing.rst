.. index::
   single: Emails; Testing

How to test that an Email is sent in a functional Test
======================================================

Sending e-mails with Symfony2 is pretty straightforward thanks to the
``SwiftmailerBundle``, which leverages the power of the `Swiftmailer`_ library.

To functionally test that an email was sent, and even assert the email subject,
content or any other headers, you can use :ref:`the Symfony2 Profiler <internals-profiler>`.

Start with an easy controller action that sends an e-mail::

    public function sendEmailAction($name)
    {
        $message = \Swift_Message::newInstance()
            ->setSubject('Hello Email')
            ->setFrom('send@example.com')
            ->setTo('recipient@example.com')
            ->setBody('You should see me from the profiler!')
        ;

        $this->get('mailer')->send($message);

        return $this->render(...);
    }

.. note::

    Don't forget to enable the profiler as explained in :doc:`/cookbook/testing/profiling`.

In your functional test, use the ``swiftmailer`` collector on the profiler
to get information about the messages send on the previous request::

    // src/Acme/DemoBundle/Tests/Controller/MailControllerTest.php
    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class MailControllerTest extends WebTestCase
    {
        public function testMailIsSentAndContentIsOk()
        {
            $client = static::createClient();
            $crawler = $client->request('POST', '/path/to/above/action');

            $mailCollector = $client->getProfile()->getCollector('swiftmailer');

            // Check that an e-mail was sent
            $this->assertEquals(1, $mailCollector->getMessageCount());

            $collectedMessages = $mailCollector->getMessages();
            $message = $collectedMessages[0];

            // Asserting e-mail data
            $this->assertInstanceOf('Swift_Message', $message);
            $this->assertEquals('Hello Email', $message->getSubject());
            $this->assertEquals('send@example.com', key($message->getFrom()));
            $this->assertEquals('recipient@example.com', key($message->getTo()));
            $this->assertEquals('You should see me from the profiler!', $message->getBody());
        }
    }

.. _Swiftmailer: http://swiftmailer.org/