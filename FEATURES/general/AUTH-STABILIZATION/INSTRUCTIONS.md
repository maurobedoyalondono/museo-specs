i have a '/home/negrito/src/projects/Museo/web' web site which works in combination with '/home/negrito/src/projects/Museo/api' to authenticate users.\
The Authentication is working more like patch that real auth system. Our goal is to analyze, design and implement high grade solution.\
\
Issues:\
For every single page in the site calls to http://localhost:8000/api/v1/auth/me are made, usually 2, pages like home do not need this at all.\ It will need only for header person name, email, and understand if there is token to display user menu.\ but calls to auth/me are absurd.\
For /orders, users should not even reach it unless they are logged in (middleware needed?)\
For pages like checkout only 1 call needs to be made to understand if the user is logged in to display the proper form\
For auth/signin, this is supposed to be the place to sign in, same as in checkout (1st step), but having errors, and I suppose the usage of general auth is terrible.\
\
Also, for pages like /orders, after they pass the middleware, i suppose we may need to mak a call to auth/me and then call the api to get the orders, what i mean is for pages that need authentication, the others calls need to be deferred until user is actually valid.\
\
You need to analyze, create a plan like '/home/negrito/src/projects/Museo/SARA-PAYMENT-PLAN.md', execute step by step.\
For the plan after each step you need to write down that it is necessary to run "npm run build" to verify the system is not broken.\
After each step update the plan.\
Plan: PLAN.md

This is about web only, the api was just for reference.

We are not here to create fast/work once solutions. We are building high class products.