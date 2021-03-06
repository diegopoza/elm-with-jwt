# Outline

- TL;DR
- Introduction
    - About Elm
        - Created by Evan Czaplicki (a JS programmer)
        - Gradual learning (the idea that you can be productive before diving deep) and usage-driven design
            - Start with Minimum Viable Solution (it's often enough; keep it simple)
            - The Elm Architecture
                - Start with simple setup: model, update, view wrapped in a module and importable 
                - Redux drew inspiration from / was inspired by the Elm Architecture
            - [Compiler errors for humans](http://elm-lang.org/blog/compiler-errors-for-humans) with helpful hints and suggestions
                - Reading error messages
                    - Lowercase values are type parameters `String -> a` (`a` is "anything could go here")
                        - `[1, 2, 3]` is type `List number` (parameterized list type only containing numbers)
                        - `[]` is type `List a` (Elm infers this is a list of anything)
        - [Functional](https://www.smashingmagazine.com/2014/07/dont-be-scared-of-functional-programming/)
            - immutable (data doesn't change, new data structures are created instead of modifying existing ones)
            - stateless (perform every task as if for the first time with no knowledge of what may have happened previously)
        - Reactive                
        - Reliable (no RTEs) - errors at compile time prevent errors at runtime  
    - We're going to authenticate with JWT
        - _TODO: this section should be extremely succinct_
- Installing Elm and tools
    - Prerequisites: Node, npm, Gulp, assumed JavaScript experience
    - How to install Elm globally: `npm install -g elm` (we're going to assume Node / npm are installed already; you will need these to run the Node API and to build the project with Gulp) 
    - How to create an Elm project: 
        - `elm package install`
    - Dependencies / build config
        - Update `elm-package.json`
        - Set up `gulp`
    - Editors / IDEs: get a plugin for syntax highlighting to avoid pain [Install](http://elm-lang.org/install)  
    - [Source repo](https://github.com/YiMihi/elm-app-jwt-api), [commits](https://github.com/YiMihi/elm-app-jwt-api/commits/master)  
- Set up Elm app
    - Create a basic main view
    - `index.html` and styles
    - Viewing the app in browser (gulp server `localhost:3000`)
- How to call an API with Elm
    - Call unauthenticated route from [nodejs-jwt-authentication-sample](https://github.com/auth0-blog/nodejs-jwt-authentication-sample) GET `/api/random-quote`
- How to authenticate with JWT
    - Register and POST to create new users `/users`
    - Log in and POST with credentials to get session token `/sessions/create`
    - Make authenticated API requests GET `/api/protected/random-quote` (use `elm-http-decorators` package to reconcile types)
    - How to log to console with `Debug.log`
    - Error handling
    - Log out (remove username, password, token, error messages)
    - Show/Hide views
    - Save model to localStorage for persisted login
- Conclusion
    - Compiler gives you a lot of test coverage "free of charge" with no RTE
    - Elm is evolving rapidly
    - Elm Architecture inspired Redux
    - Culture and resources to learn more
        - Slack  
        - NoRedInk (SF) using Elm in production for 8 months
            - 95% of front-end programming done in Elm with no runtime exceptions (Richard Feldman 3-22-16)
            - Easy to maintain and refactor
            - Compiler catches things that tests don't because maybe you didn't think to write a test for that
        - Styleguide (layouting, whitespace, chunking, newlines) [Styleguide](http://elm-lang.org/docs/style-guide) 

## Steps Outline

1. Install Elm, play with `elm-repl`
2. Set up build (`gulpfile.js`, `package.json`, `elm-package.json`, `npm install`)
3. Step 1 - `program` with model record that updates on button click, stylesheets, and CSS in view
4. Step 2 - `HTTP` GET unauthenticated quote and display in view with styles
5. Step 3 - `HTTP` POST to register, authbox view, decode token, error messaging, debug log
6. Step 4 - `HTTP` POST to log in, build on register code, add log out
7. Step 5 - `HTTP` GET authenticated quote, add decorators package for type reconciliation, clean up logs
8. Step 6 - `programWithFlags` Save model to `localStorage`  _(Step6.elm == Main.elm)_

## Familiarizing with Elm

_TODO (important): where applicable, these points should go along with the steps they are applicable to; reassign to steps in article body_

- Note about 0.16 -> 0.17(+) breaking changes, make note of this fact when referencing any resource material other than docs
- `elm-repl` (Read Eval Print Loop) in command line
- MODEL 
    - Represents the current application state
    - Record - similar to objects in JS (but with key differences)
    - A record is immutable data - no inheritance, no methods
    - Use persistent data structures to return a new model that is a copy of the old model with the updated data efficiently (only really copies the part that changed)
    - Record update syntax: `{ model | property = newValue }`
- UPDATE 
    - Transitions between application states 
    - Case expressions are indentation sensitive   
- VIEW 
    - Describes the rendered view based on the application state 
    - Uses virtual DOM to render: view is a function and corresponds to DOM nodes, the nodes are actually functions that pass lists as parameters 
    - Every time model changes, view function runs again and diffs the previous virtual DOM with the next and runs the minimal set of updates necessary
- Instead of callbacks, messages respond to user interaction - these update the model
- `let` in Elm is like constants in JS, they cannot be reassigned like variables
- Type annotation syntax 
    - `type alias Model`, `model : Model`
    - `type` vs `type alias`
        - `type`: creating a completely new type [Types](http://guide.elm-lang.org/types/)
        - `type alias`: interchangeable with a record of this type (provides more detail in one place while being less verbose when referencing later) [Type Aliases](http://guide.elm-lang.org/types/type_aliases.html)
    - `String -> String -> Int -> String` "function takes two strings and a number and returns a string"
        - Currying: if you don't pass all the arguments, it gives you back another function that accepts whatever arguments are still needed _(more on currying elsewhere, don't go into too much detail here)_
            
        ```
        function2 : Int -> String
        function2 = function1 "string" "string"

        function2 : String -> String -> Int -> String
        function2 str1 str2 someInt =
            if someInt > 1 then
                str1
            else
                str2
        ``` 
    - Can also pass something like `foo -> foo -> String` and `foo` could be any type, but must be consistently the same type _(exclude this?)_
    - Top-level things should have type annotation, but can use them everywhere (may help beginners) 
    - [How to Read a Type Annotation](https://github.com/elm-guides/elm-for-js/blob/master/How%20to%20Read%20a%20Type%20Annotation.md)
- `Model -> (Cmd Msg)` takes a model and returns a command that accepts a message parameter  
- Union types: case expression 
    - Model the user interactions and can have a guarantee of no RTE for all branches
    - Type parameters have to be consistent
- Anonymous functions 
    - Anonymous function with argument: `\str -> "Hi " ++ str`  
    - Anonymous function that discards its argument: `\_ -> "Hi"` 
    - `\` is meant to look like a lambda, which is the symbol for function in functional programming
- `http` are effects: descriptions of what you want done, but don't execute until handed off to Elm runtime
- `Debug.log` type annotation is `String -> a -> a`
    - `Debug.log "the value of a is" a`
    - `thing |> Debug.log "thing is"` or `(Debug.log "thing is" thing)`   
- `|>` and `<|` are aliases for function application
    - this helps to reduce parentheses
    - [<|](http://package.elm-lang.org/packages/elm-lang/core/4.0.1/Basics#%3C|) backward function application
    - [|>](http://package.elm-lang.org/packages/elm-lang/core/4.0.1/Basics#|%3E) forward function application

## Considerations / Research

- [x] Gulp
- [x] Bootstrap
- [x] Elm: HTTP POST requests / response handling
- [x] How to show/hide different views
- [x] Error handling
- [x] Port localStorage into Elm from JavaScript and utilize [#8](https://github.com/YiMihi/elm-with-jwt/issues/8)

## Collected Resources

- Install the Elm language binaries: [https://www.npmjs.com/package/elm](https://www.npmjs.com/package/elm) `npm install -g elm`
- Elm tools [Get Started](http://elm-lang.org/get-started)
- [Documentation](http://elm-lang.org/docs) / [The Guide](http://guide.elm-lang.org/)
- [Tutorial](http://www.elm-tutorial.org/en)
- [HTTP example](http://elm-lang.org/examples/http)
- [gulp-elm](https://www.npmjs.com/package/gulp-elm)
- [Extracting results of HTTP requests in Elm](http://stackoverflow.com/questions/35028430/how-to-extract-the-results-of-http-requests-in-elm)
- [How to decode data from JSON](http://stackoverflow.com/questions/32575003/elm-how-to-decode-data-from-json-api)
- [elm-http-decorators interpretStatus](http://package.elm-lang.org/packages/rgrempel/elm-http-decorators/1.0.2/Http-Decorators#interpretStatus)
- [elmlang Slack](http://elmlang.herokuapp.com)
- [Error handling SO](http://stackoverflow.com/questions/37390998/how-can-i-get-the-error-message-out-of-http-error)
- Video: [Let's be mainstream! Evan Czaplicki](https://www.youtube.com/watch?v=oYk8CKH7OhE)
- Video: [Introduction to Elm - Richard Fedlman](https://www.youtube.com/watch?v=zBHB9i8e3Kc)
- Video: [Elm Basics](https://www.youtube.com/watch?v=g48K6ABfRzA)
- [Styleguide](http://elm-lang.org/docs/style-guide)
- [Elm Destructuring](https://gist.github.com/yang-wei/4f563fbf81ff843e8b1e)
- [Elm articles by Dennis Reimann](https://dennisreimann.de/articles/elm.html)

### Notes

`Http.string <| Encode.encode 0 <| userEncoder model` Breakdown:

- `userEncoder model` produces a json `Value`
- `Encode.encode` formats that as a string. The first argument is an int to indicate how much indentation to use for pretty printing the JSON; with `0` it condenses the output as much as possible