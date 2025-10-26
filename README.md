## Mevrik Channels PHP – Omnichannel Messaging Platform (BlueBird)

Overview

The Mevrik Channels PHP repository powers a complete digital customer experience platform. It seamlessly integrates multiple communication channels—website widgets, Facebook Messenger, email, and social media comments—allowing businesses to engage customers efficiently and consistently. The system includes an AI-powered bot service using Dialogflow, enabling automated customer interactions. If the bot is unable to handle a customer’s request, it automatically triggers a “Connect Agent” option, ensuring smooth handover to human agents. By centralizing message processing, agent responses, case management, CSAT collection, and reporting, it delivers a fully integrated solution that enhances customer satisfaction and drives superior digital experiences.

Key Features

Multi-Channel Integration: The service supports a wide array of communication channels, including:

Widgets: Embedded chat interfaces on websites.

Facebook: Integration with Facebook Messenger platform and Facebook social platform.

Signline: A built-in video calling channel that allows real-time face-to-face interaction between customers and agents directly within the platform.

Email: Handling of email communications.

Webhook Processing: The system listens for incoming webhooks from platforms like Facebook, processes the data, and triggers appropriate actions within the system.

Agent Interaction: Customer service agents can respond to customer messages through the system's unified interface, ensuring consistent and efficient communication.

Customer Satisfaction (CSAT) Surveys: After resolving a conversation, the system sends out CSAT surveys to customers, allowing businesses to gauge customer satisfaction and improve service quality.

Case Closure: Once a conversation is concluded and the CSAT survey is sent, the case is marked as closed, streamlining case management.

Reporting: The system generates comprehensive reports, providing insights into communication metrics, agent performance, and customer satisfaction trends.

Technical Stack :

Programming Language: PHP8.2

Framework: Laravel

Database: MySQL or PostgreSQL, Redis, Clickhouse

External Integrations: Facebook Graph API, Email APIs

Usage

To utilize the Mevrik Channels PHP service:

Clone the Repository:

git clone https://github.com/genexgit/mevrik-channels-php.git


Install Dependencies:
Navigate to the project directory and install the required PHP packages:

cd mevrik-channels-php
composer install


Configuration:

Set up environment variables in the .env file for API keys, database credentials, and other configurations.

Configure webhooks for platforms like Facebook to point to the appropriate endpoints in the service.


Start the Service:
Run the service using Laravel's built-in server or configure it with a web server like Nginx