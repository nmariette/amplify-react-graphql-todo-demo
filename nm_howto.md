
# Notes on how to get a Graph QL React demo app working

Initial reference: https://mokkapps.de/blog/build-and-deploy-a-serverless-graphql-react-app-using-aws-amplify/

Github reference: https://github.com/nmariette/amplify-react-graphql-todo-demo

This all looked so easy! I cloned the github and went straight into the folder and followed instructions, but it didn't work. I tried some other things and got around issues but it was only partly working. Then I went back to the mokkapps page and tried going from the raw instructions there. This also didn't work as I ended up with nested versions of the app, which amplify didn't like. However I also made progress because just cloning the repo wasn't enough to get an AWS Amplify account going (of course). So then I went back to the repo instructions and worked on overcoming the errors there. It was at this stage that I decided I should start documenting my process. What follows is described below:

When running `yarn start`, I get an error: `Package subpath './lib/tokenize' is not defined by "exports"`

This page and others suggest that I need to use LTS version of node: https://github.com/mifi/lossless-cut/issues/971 and a fix may be to run `yarn upgrade`. That doesn't work though. This page then suggests using `node -v` to determine version, then install a stable version using `n`. https://stackoverflow.com/questions/47008159/how-to-downgrade-node-version

```
node -v // find out version
npm -v  // same
sudo npm install -g n   // install n
sudo n  stable
node -v
```

This took me from v19.2.0 (not stable version) to v18.12.1 (LTS stable as confirmed by https://nodejs.org/en/)

Now back to instructions here: https://github.com/nmariette/amplify-react-graphql-todo-demo

Yes! It starts without error, but then I get this in my browser:

> This site can’t be reached  
> localhost refused to connect.  
> Try:
>
> Checking the connection  
> Checking the proxy and the firewall  
> ERR_CONNECTION_REFUSED

Now when I run `yarn start` I get `Error: error:0308010C:digital envelope routines::unsupported`

https://www.freecodecamp.org/news/error-error-0308010c-digital-envelope-routines-unsupported-node-error-solved/ suggests three possible solutions:

1. a command option to react-scripts
2. use LTS Node JS (already using 18.12.1 which is LTS), but suggests downgrading even to 16.16.0. (don't like having to go earlier than current LTS)
3. upgrade react script to v5+. I'll try this.

```
npm uninstall react-scripts
npm install react-scripts   \\ got some warnings
npm audit fix
npm audit fix --force       \\ got errors
\\ lots of stuff not working. Tried various stuff in here
rm -rf node_modules
rm package-lock.json
yarn
yarn start
```

This started something that showed lots of errors. E.g.:

>Compiled with problems:X
>
> ERROR in ./node_modules/@aws-amplify/pubsub/lib-esm/ Providers/AWSAppSyncRealTimeProvider.js 311:36-48
>
> export 'GraphQLError' (imported as 'GraphQLError') was not found in 'graphql' (module has no exports)

Seems like GraphQL problems, so back to the mokkapps page, where there's a section on creatin the GraphQL API:

```
amplify add api
```

Nope. Stopped because this was starting down a route of complication with AWS - asking for the API key again, etc. Stuff

Then tried suggestions on this page (found using search: "create react" <error text>) https://github.com/aws-amplify/amplify-js/issues/4387 and this page https://docs.amplify.aws/start/getting-started/setup/q/integration/js/
Added webpack config and put config in it from above two pages. Didn't fix anything.

So going back to the AWS link immediately above, installing amplify again. Getting lots of deprecated warnings. Have to go out soon for kids birthday party, so it may be a fail for now.

Tried reinstalling amplify: `npm install aws-amplify`, then `yarn start`, and got back to

> localhost refused to connect

tried `npm install webpack webpack-cli webpack-dev-server copy-webpack-plugin --save-dev`, got errors.

Maybe there are issues with using two package managers: yarn and npm?

tried `yarn start`, got errors. Following instructions from output:

```
1. Delete package-lock.json (not package.json!) and/or yarn.lock in your project folder.
2. Delete node_modules in your project folder.
3. Remove "webpack-dev-server" from dependencies and/or devDependencies in the package.json file in your project folder.
4. Run npm install or yarn, depending on the package manager you use.

In most cases, this should be enough to fix the problem.
If this has not helped, there are a few other things you can try:

5. If you used npm, install yarn (http://yarnpkg.com/) and repeat the above steps with it instead.
   This may help because npm has known issues with package hoisting which may get resolved in future versions.

6. Check if /Users/nickm/nm_dev_2022/amplify-react-graphql-todo-demo/node_modules/webpack-dev-server is outside your project directory.
   For example, you might have accidentally installed something in your home folder.

7. Try running npm ls webpack-dev-server in your project folder.
   This will tell you which other package (apart from the expected react-scripts) installed webpack-dev-server.

If nothing else helps, add SKIP_PREFLIGHT_CHECK=true to an .env file in your project.
That would permanently disable this preflight check in case you want to proceed anyway.
```

Still getting refused to connect. Read https://www.freecodecamp.org/news/error-error-0308010c-digital-envelope-routines-unsupported-node-error-solved/ and tried adding `react-scripts --openssl-legacy-provider start` to `package.json`. Then had to reverse the comments I put around the following lines of /src/App.js:

```
import { withAuthenticator } from '@aws-amplify/ui-react';
export default withAuthenticator(App);
```

And then.... it worked!!!
