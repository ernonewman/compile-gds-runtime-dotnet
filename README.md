## How to Import the GOV UK Design System into a .NET MVC Project

_The walkthrough will also detail how to set-up your project to compile the sass at runtime_

**author**: Nathan Ouriach

**peer review**: Rob Marshall

___

### Installing the correct npm packages

- Firstly, ensure you have [Node](https://nodejs.org/en/) installed 
- Open up your terminal and navigate to the root of your project folder
- Once inside the root of your project folder run the command `npm init`
- Then install the correct dependencies by running the following commands
```sh
  npm install sass
  npm install dart-sass
  npm install govuk-frontend --save
```
___
### Updating your project

- Add the following to your `.gitignore`:
```
# CSS files (because we're using SASS)
*.css
*.css.map

# govuk frontend stuff
[full path from root of your project]/wwwroot/assets
[full path from root of your project]/wwwroot/js/govuk.js

# minified js
*.min.js
```
- Then inside your .NET MVC project find the `wwwroot/css` folder create a `main.scss` file
- Add the line `@import "node_modules/govuk-frontend/govuk/all";` to `main.scss`
- Inside your `_Layout.cshtml` file you will need to add the following code to the bottom of your `<head>` element, this will reference your new `main.css` file

```html
  <head>
    //
    <link rel="stylesheet" href="~/css/main.css" asp-append-version="true" />
  </head>
```
___
### Setting up the Javascript

- Inside your `_Layout.cshtml` file  paste the following code into the top of your `<body>` element
```html
  <body>
    <script>document.body.className = ((document.body.className) ? document.body.className + ' js-enabled' : 'js-enabled');
    </script>
    //
  </body>
``` 

- The compile steps below will create a new `govuk.js` file
- Import this file before the close `</body>` element inside your `Layout.cshtml` file and then run the `initAll()` function to initialise all the components.

```html
  <body>
      <script>document.body.className = ((document.body.className) ? document.body.className + ' js-enabled' : 'js-enabled');
      </script>
        //
        //
      <script src="~/js/govuk.js" asp-append-version="true"></script>
      <script>
        window.GOVUKFrontend.initAll()
      </script>
  </body>
```
___
### Compile the .sass file

- Locate your `package.json` file and add the following script
```json
"scripts" : {
    "compile-sass" : "dart-sass .[full path from root of your project]/wwwroot/scss/main.scss [full path from root of your project]/wwwroot/css/main.css"
 } 
```
- This tells the compiler to target a specific sass file and render it into a specific css file
- In the command line rune - and in the same location as your `package.json` file - run the command: `npm run compile-sass`
- To test it has worked, select a random component from the [GOV UK Design System](https://design-system.service.gov.uk/components/) and paste the HTML into one of your `.cshtml` View files
- Run the project and then you should see the chosen component rendered correctly
___
### How to compile the sass using gulp
- Install gulp and required packages:
```sh
npm install gulp gulp-clean-css gulp-dart-sass gulp-rename gulp-uglify
```

- In the root of your folder create a new file called `gulp.js` (this will exist at the same level as your `Startup.cs` file)
- At the top of your new `gulp.js` file import the above dependencies, your code will look like the below

```js
var sass = require("gulp-dart-sass");
var async = require("async");
var rename = require("gulp-rename");
var cleanCSS = require('gulp-clean-css');
var uglify = require('gulp-uglify');
// //
```

- You will then create a function underneath these imports that will build the sass by grabbing all of your projects `.scss` files  and directing them to your new `main.css` file. Your function will look like the below:
```js
const buildSass = () => gulp.src("wwwroot/css/*.scss")
	.pipe(sass().on("error", sass.logError))
	.pipe(cleanCSS())
	.pipe(gulp.dest("wwwroot/css"));
```
You will also need a function to copy over the assets from the _node_modules_ folder and place them within your _wwwroot/assets_ folder, as well as bringing over the `.js` file contents into your project's `govuk.js` file:
```js
const copyGovukAssets = () => gulp.src(["node_modules/govuk-frontend/govuk/assets/**/*"]).pipe(gulp.dest("wwwroot/assets")).on("end", () =>
	    gulp.src(["node_modules/govuk-frontend/govuk/all.js"]).pipe(rename("govuk.js")).pipe(gulp.dest("wwwroot/js/")));
	    gulp.src(["node_modules/govuk-frontend/govuk/all.js"]).pipe(uglify()).pipe(rename("govuk.js")).pipe(gulp.dest("wwwroot/js/")));
```
Close your `gulp.js` file by creating a task that runs the above functions
```js
gulp.task("build-fe", () => {
	return async.series([
		(next) => buildSass().on("end", next),
		(next) => copyGovukAssets().on("end", next)
	])
}); 
```
You can now build the front end and copy over the assets using `npx gulp build-fe`.
___
### Add the gulp tasks to the `package.json` file to run at compile time
- Locate your `package.json` file and add the following script 
- By adding the below script you will be able to run the gulp task with `npm run build` during the deployment process
```json
{
    //
    "scripts": {
            "build": "gulp build-fe"
    }
    //
}
```
