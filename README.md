# Project Documentation/Report Outline

- ### Title: Content Management System
- ### Author(s): Collin Edwards
- ### Date: Nov 2023 - Jan 2024

### Introduction

Our team's localized content currently exists within a mostly-inaccessible (at the very least inconvenient) repository. Many of [our team]-adjacent teams and services have requested a means through which they can access this content. Additionally, UX has expressed interest in having more agency in managing the content (it is currently gated behind dev-specific actions). To solve these needs, we have proposed a centralized service which will eventually utilize AEM connectivity to retrieve and construct localized content.

### Requirements

- The service will be easily accessible by internal teams
- The service will provide a myriad of unique content keys and values in such a way that has a strict response definition
- The service will have auth in place
- The service will initially house the content within itself, but later migrate to storing the content in Adobe Experience Manager
- The content file structure will be easily ported to AEM

### Planning

Before any work had begun, I created an overview document in confluence, and presented what I thought the ideal routes were as far as design and implementation. This included a number of architecture and data flow diagrams, as well as a draft API contract with schema examples. We discussed the options as a team and later modified the document to reflect our consensus.

### Process

The first step of creating this service was to submit an official API contract to outline the service's endpoints and functionality. This required meeting with the senior dev on the team initially, and later with the principal dev, who had very valuable feedback on the contract. At this stage, I learned the importance of constricting the request/response definitions, and the possible negative impact poor API design can have on clients. We eventually finalized the definition, and moved forward to the service's implementation. For the language, I decided on Java, since the majority of our backend services were using that language, and I thought it would be easier for other team members to onboard and contribute if desired. My first PRs were related to the initial construction of the defined endpoints (which came along quite smoothly), and the migration/restructuring of our content over from a separate repo. Considering there were thousands of localization files that needed adjusting, I opted to write a python script to handle this. This script took the .properties content files, iterated through each file/locale and mapped the keys and values to JSON, while modifying the key values to match our new key structure. It was fun to go back to python, since I hadn't used it for about a year! After these changes got merged, the next step was to set up the deployment pipeline--AWS via ECS. I had done a fair amount of CICD work up until this point, so it was super simple. The following PR addressed the i18n portion of processing content, where I had to create a mapper with pluralization capabilities in order to retain our previous systems functionality. This part was a bit tricky, as there are a lot of catch-points where exceptions could arise. I made use of the common ICU4J library to make this process smoother. In consideration of human error and other edge cases, I opted to enable the capability to fall back on default/static content in cases where content could not be retrieved, or variables were missing. After this was successfully implemented, it was time to integrate our internal authentication to the service. This aspect of the project was particularly frustrating, as I found out later our other services exist in a secure environment and lack the ability to generate the necessary in-house token to connect. I spent a couple weeks troubleshooting but ultimately had to create a workaround using Basic authentication for these specific services (making use of Hashicorp Vault). At this point, the M1 work was complete, although I am still working across repos to move toward additional milestones.

### Testing

Since this was all net-new code, unit tests and integration tests had to be written to ensure at least 90% coverage. This was a fairly straightforward process, but I deviated a bit from the standard and decided to write the tests in kotlin--I loved it! I'm still not totally sold on building applications solely using kotlin but for mocking data and verbosity (lack thereof), kotlin is very ideal for writing tests. Other than that, I used a pretty standard suite of Mockito for unit tests, and MockMvc for integration tests.

### Maintenance

While we are moving toward later milestones, the content within the service needs to be frequently kept up-to-date using the previously mentioned python script.

### Outcomes

Currently, the CMS is functional and deployed, but not in-use, as there is work that needs to be done across other services to start utilizing it. The migration to Adobe Experience Manager will occur later as well. Overall, I learned a ton about API design/architecting as well as how implementations can change them. This was a wonderful project and I'm glad I got the opportunity to lead on a feature many other teams have been requesting.
