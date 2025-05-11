[Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) is a powerful tool for monitoring the performance and usage of your applications. It provides insights into how users interact with your app, identifies areas for improvement, and helps you understand the impact of changes.

- [Users, Sessions & Events](https://learn.microsoft.com/en-us/azure/azure-monitor/app/usage?tabs=aspnetcore#users-sessions-and-events---analyze-telemetry-from-three-perspectives) - Track and analyze user interaction with your application, session trends, and specific events to gain insights into user behavior and app performance.
	
- [Funnels](https://learn.microsoft.com/en-us/azure/azure-monitor/app/usage?tabs=aspnetcore#funnels---discover-how-customers-use-your-application) - Understand how users progress through a series of steps in your application and where they might be dropping off.
	
- [User Flows](https://learn.microsoft.com/en-us/azure/azure-monitor/app/usage?tabs=aspnetcore#user-flows---analyze-user-navigation-patterns) - Visualize user paths to identify the most common routes and pinpointing areas where users are most engaged users or may encounter issues.
	
- [Cohorts](https://learn.microsoft.com/en-us/azure/azure-monitor/app/usage?tabs=aspnetcore#cohorts---analyze-a-specific-set-of-users-sessions-events-or-operations) - Group users or events by common characteristics to analyze behavior patterns, feature usage, and the impact of changes over time.
	
- [Impact Analysis](https://learn.microsoft.com/en-us/azure/azure-monitor/app/usage?tabs=aspnetcore#impact-analysis---discover-how-different-properties-influence-conversion-rates) - Analyze how application performance metrics, like load times, influence user experience and behavior, to help you to prioritize improvements.
	
- [HEART](https://learn.microsoft.com/en-us/azure/azure-monitor/app/usage?tabs=aspnetcore#heart---five-dimensions-of-customer-experience) - Utilize the HEART framework to measure and understand user Happiness, Engagement, Adoption, Retention, and Task success.

## Users, Sessions, and Events - Analyze telemetry from three perspectives

- **Users tool**: How many people used your app and its features? Users are counted by using anonymous IDs stored in browser cookies. A single person using different browsers or machines will be counted as more than one user.
    
- **Sessions tool**: How many sessions of user activity have included certain pages and features of your app? A session is reset after half an hour of user inactivity, or after 24 hours of continuous use.
    
- **Events tool**: How often are certain pages and features of your app used? A page view is counted when a browser loads a page from your app, provided you've [instrumented it](https://learn.microsoft.com/en-us/azure/azure-monitor/app/javascript).

### Explore usage demographics and statistics
- The **Users** report counts the numbers of unique users that access your pages within your chosen time periods. For web apps, users are counted by using cookies. If someone accesses your site with different browsers or client machines, or clears their cookies, they're counted more than once.
    
- The **Sessions** report tabulates the number of user sessions that access your site. A session represents a period of activity initiated by a user and concludes with a period of inactivity exceeding half an hour.

#### How many Application Insights resources should I deploy?
When you're developing the next version of a web application, you don't want to mix up the [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) telemetry from the new version and the already released version.

To avoid confusion, send the telemetry from different development stages to separate Application Insights resources with separate connection strings.

#### When to use a single Application Insights resource
- Streamlining DevOps/ITOps management for applications deployed together, typically developed and managed by the same team.
- Centralizing key performance indicators, such as response times and failure rates, in a dashboard by default. Segment by role name in the metrics explorer if necessary.
- When there's no need for different Azure role-based access control management between application components.
- When identical metrics alert criteria, continuous exports, and billing/quotas management across components suffice.
- When it's acceptable for an API key to access data from all components equally, and 10 API keys meet the needs across all components.
- When the same smart detection and work item integration settings are suitable across all roles.