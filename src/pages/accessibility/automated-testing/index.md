---
title: Automated Accessibility Testing Tools
---
## Automated Accessibility Testing

This is a stub. <a href='https://github.com/freecodecamp/guides/tree/master/src/pages/accessibility/automated-testing/index.md' target='_blank' rel='nofollow'>Help our community expand it</a>.

<a href='https://github.com/freecodecamp/guides/blob/master/README.md' target='_blank' rel='nofollow'>This quick style guide will help ensure your pull request gets accepted</a>.

<!-- The article goes here, in GitHub-flavored Markdown. Feel free to add YouTube videos, images, and CodePen/JSBin embeds  -->

#### More Information:
<!-- Please add any articles you think might be helpful to read before writing the article -->

#tl;dr

For code base testing, there are some good tools for JavaScript and React. But for a good overview of the errors we must test on a generated DOM; aXe and pa11y perform well here.

Currently, manual testing in the browser is still necessary, particularly for keyboard navigation and screen reader feedback on dynamic changes. And unfortunately, there is no single big button to catch all errors in one report.

#Scope

This research is about integrating automated accessibility testing during development using npm modules, command line, and other tools.

#Out of scope

Testing in the browser is already pretty well covered by browser addons like Axe, HTML_CodeSniffer, and the Accessibility Inspector in the Firefox Developer Tools and Chrome Dev Tools.

Axe adds a tab to the inspector and HTML CodeSniffer adds a bookmarklet that displays a popup with the errors and warnings.

The W3C has online developer tools like the Markup Validation Service and the CSS Validation Service to validate the HTML and CSS of the frontend of your work.

#Expectations

As I mentioned previously, there is no single script that catches all accessibility errors in a workable report for a whole project in one go. That would be nice!

Spoiler alert: that’s currently impossible. But we can do a lot of automated testing already.

The Government Digital Service did an Accessibility tool audit, which includes information on what can be tested and the performance of the tools available for testing. Not all listed tools are open source or in-browser based.

#Tools and workflow

The recommended workflow for accessibility testing is currently like this:

check the code base
check the DOM
check the keyboard navigation.
The first can be automated, the second only partly, and the third still has to be done manually during development.

The difference between testing for PHP and JavaScript errors is that the accessibility needs to be tested on a generated DOM, including the different “responsive” views. This accounts for heading structure, colour contrast between text and background, generated content by JavaScript, and screen reader feedback on dynamic changes.

##Tools to test the code base

Some JavaScript check modules can be included in your test routine to test the code base.

Here are a couple I recommend:

eslint-plugin-jsx-a11y by Ethan Cohen. Static AST checker for a11y rules on JSX elements.
react-a11y by ReactJs. Warns about potential accessibility issues with your React elements.
For a project within WordPress it’s impossible to check the HTML for semantic errors from the PHP code base, because most of the HTML is generated by the PHP. This makes it hard to get an overview of what is eventually generated and how this relates to other functionality.

##Tools to test the DOM

At the moment there are two prominent CLI modules that can create a DOM from a url and perform accessibility tests on them.

pa11y by Rowan Manning
aXe-cli by Deque Labs
pa11y runs HTML CodeSniffer from the command line for programmatic accessibility reporting.

###Instructions for use

Install pa11y for CLI:

```npm install -g pa11y```
Then run pa11y in the command line:

```pa11y your-url```
aXe-cli runs axe-core from the command line. It default runs headless Chrome to generate an instance of the DOM.

Install axe-core for CLI:

```npm install axe-cli -g```
```npm install chromedriver –g```
Then run axe in the command line:

axe your-url
Both modules are highly configurable to meet your test requirements. Personally I think that aXe generates better error warnings and pa11y is easier to configure.

On the pa11y GitHub repository Rowan Manning proposed researching the possibility of replacing HTML CodeSniffer by aXe-core, so that’s good news.

The typical way to work with CLI tools like this is to run them for one url at the time. That way you get a readable report of all the errors on that page.

More than one url at the time?
You can run ```axe-cli``` on more then one url in one command, but axe-cli isn’t built to run on a large amount of urls or on a complete site; axe-cli is not a crawler. Deque Labs recommends using the  axe-webdriverjs, a chainable aXe API for Selenium’s WebDriverJS, in order to test on a large number of urls.

But do you want to test every url in a project during development? For one, you will get many duplicate errors. For example, if there is an error in the header, it will be reported for every page. Secondly, if you run a test on all pages in your project it will take long time to generate a report. And if you work with a team, all the errors in your team members work will also be reported.

If you include all urls possible in your project, it will result in slow, unreadable reports. So adding this to your Grunt / Gulp routine may not be advisable.

As an alternative, you may want to generate a list of possible templates, and define some mapping instead of running everything at once. You can start by testing a page, a post, a custom post type page, an archive, a contact page, or a page with a custom template. That way you minimise the risk of duplication and the regeneration time, with the likelihood of receiving a more usable report.

When you use Pattern Lab it’s easy to include the pages with the different components.

For example, like this in your Grunt file:

shell: {
    axe: { command: () => 'axe http://your-site.local/patternlab/public/patterns/01-molecules/index.html, http://your-site.local/patternlab/public/patterns/02-organisms/index.html -b c' },
    phpcs: { ... },
    phpunit: {... }
 },
In my experience, running axe in the command line before a commit is the easiest, fastest, and most accurate method of achieving this.

WordPress trunk
What about WordPress trunk?

Can we use automated a11y testing, included for the code standard checks?

PHPCS looks at the raw code (it doesn’t parse it as PHP, only as individual tokens), so isn’t generally suitable for creating sniffs for accessibility tests.

Automated accessibility testing needs to be on a fully working server-driven site (even if it’s local), and not just a collection of files.

A bash script could be written that does the following:

Pulls down WordPress, or uses an existing local install,
Sets up a database with some demo content (much like the Unit Tests structure does),
Optionally takes a diff and applies it to the WordPress codebase,
Runs aXe over the resulting site.
This bash script could then be hooked into a SVN/Git pre-commit hook, so that a failure in aXe would halt the commit.

Halting a commit on an aXe failure is kind of tricky. aXe gives a return status but does not distinguish between errors or warnings. Also, aXe (or any other a11y check tool) gives false positives. Manually checking the results remains a necessary part of the process.

Note: keyboard testing and screen reader feedback on dynamic changes still needs to be performed manually.
