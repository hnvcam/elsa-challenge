# System Scope
- Support users accessing by web browser.
- CCU < 10.000
- The content of each type of quiz is fixed, NO dynamic quiz creation.

# Sample UI
![/UI Design.png](https://github.com/hnvcam/elsa-challenge/blob/main/UI%20Design.png)

- By default, users are not required to log in to the system. This allows for a more casual and accessible experience, encouraging users to explore the challenges without feeling pressured to commit.
- Based on anonymous or authenticated session data, we display a user's history, including their score and the number of quizzes completed.
- The leaderboard is automatically updated with the latest data to provide a real-time view of rankings."

# High level design
![High-level design](https://github.com/hnvcam/elsa-challenge/blob/main/Esla%20Tech%20Challenge-High-level.png)

### Chosen Approach: Serverless Architecture with Static Page Generation (SSG)
##### PROS
- **Rapid Deployment**: Serverless architecture and SSG enable quick and efficient releases, reducing time-to-market.
- **Enhanced SEO**: SSG generates static HTML files, which are highly optimized for search engines, improving website visibility.
- **Simplified Management**: Eliminates the complexities of server management, allowing teams to focus on core development and functionality.
- **Scalability**: Serverless functions automatically scale based on demand, ensuring optimal performance and cost-efficiency.
- **Cost-Effectiveness**: Pay-as-you-go pricing models in serverless environments significantly reduce infrastructure costs, particularly for short-term campaigns, infrequent access applications, or applications with unpredictable traffic patterns.

##### CONS
- **Limit for Dynamic Content**: It is required to utilize other technique like Server-side rendering (SSR) or client-side data fetching in order to provide dynamic content.
- **Vendor Lock-in**: When chosing a serverless infrastructure, it also introduces the challenges to migrate to another platform.
- **Debugging Challenges**: Troubleshooting issues in serverless environments can be more complex due to the distributed nature of the architecture.
- **Concurrency Limits**: Serverless functions might have limitations on the number of concurrent executions, which can impact performance under heavy loads.

### The generic flows of the system in high-level
- All quizzes will be created and managed using a headless CMS, such as Contentful or Storyblok. This eliminates the need for extensive upfront development work to design and implement a custom backend system.
By utilizing a headless CMS, we can focus on building a robust and user-friendly frontend experience. Additionally, the CMS provides a comprehensive set of APIs for accessing and managing quiz content, saving us the time and resources required to develop our own custom content management solution.
- During the CI/CD build process, we will retrieve all quiz data from the headless CMS to populate the quiz list on the homepage and implement pagination for the all-quizzes page.
Dynamic page generation will also create the presentation of individual quizzes, including all questions and answer options. It's essential to ensure that the correct answers are not revealed in the frontend presentation; this logic should be handled by cloud functions.
- When a user browses to the application, we will implement anonymous authentication using Firebase or restore previous logged-in sessions. This allows us to track user progress without requiring explicit logins, while also ensuring backend security.
- Upon completing a quiz, the client-side application (using Redux and Redux Saga) will submit the user's selected answers (including question IDs and selection IDs) to a cloud function.
The cloud function will then query the correct answers for the corresponding questions using GraphQL. It will then calculate the user's score based on the accuracy of their answers.
Next, the cloud function will update the user's state data and compare it with the leaderboard data to determine if the user should be added or removed from the leaderboard.
Leveraging the real-time update capabilities of Firebase Firestore, the client-side leaderboard component will be automatically notified of any changes to the leaderboard data. This eliminates the need for implementing additional real-time communication technologies like WebSocket or WebRTC

# Modules design
![module-design](https://github.com/hnvcam/elsa-challenge/blob/main/Esla%20Tech%20Challenge-Modules%20Design.png)

| System | Module | Description |
| --- | --- | --- |
| Client | Login module | This React component set will do the anonymous login or providing a login screen to user. Firebase does support various types of login and react component to integrate with. |
| | Home page module | This page includes other modules (React components) to display data to users. There are 2 parts in the home page: <br/>- Static: The list of Quizzes by SSG <br/>- Dynamic: The User state and Leaderboad by api request and Firestore real-time update |
| | User state module | This page is simply a React component that displays the user score together of number of taken quizzes from the query to Firestore by user Id. |
| | Leaderboard module | This is also a React component that queries data from Firestore and display it. It also subscribes for changes to these objects. |
| | Quizzes pages | These pages are generated based on the data from Headless CMS. With most of SSG frameworks like Gatsy, NextJS, Astro, pagination is also supported. |
| | Test taking modules | These are static pages with form submission to the Firebase HTTPs endpoint. |
| Server | Firebase Firestore | This services provide content storage for documents and queries. We will store the User State, Leaderboard, and AnswerSubmission on this document storage |
| | Firebase Functions | Provides the HTTPs endpoint to accept AnswerSubmission and query the correct answer from Headless CMS, then calculate the score. This function also update to User State and Leaderboard |
| | Headless CMS | Provides the CMS functionality with prebuilt UI for back office. |
| SSG | Gatsby Dynamic pages | Retrieves the data from Headless CMS to build Quizzes page and Test page content. This will be executed by the CI. |

  
