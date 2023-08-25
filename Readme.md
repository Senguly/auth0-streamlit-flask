## Application structure

There are three parts to my solution:

1. __Streamlit component host app__ (localhost, port 4009)
2. __Next.js (React) component__ (localhost, port 3001)
3. __Flask server hosting API endpoint__ (localhost, port 8888)

## [1] Streamlit component host app

This uses a component design similar to [@synode's](https://discuss.streamlit.io/u/synode/summary) [Streamlit-Player](https://discuss.streamlit.io/t/streamlit-player/3169) and generalises its ability to receive `OnStatusUpdate` and `OnError` _events_ from the hosted component, so it can be used more easily as a component template in other applications. The generalisation includes an example of handling a "delegated host action" `OnActionRequest` event. `auth0_login_component` component provided is able to handle `OnActionRequest` where the component can request the host (i.e. the Streamlit app) to make a web request.

## [2] Next.js (React) component

The component implementation is in `/pages/streamlit` and similar to the one described in `Streamlit Component-template`, reference 2, with some extras motivated by the `ReactPlayer integration` in reference 3.

The component UI is minimal and simply displays the user's authentication status. It's primary job is to listen for `window.localstorage` changes in an authentication `jwt` token, and send these as `OnStatusUpdate` events to the Streamlit host application, which can act accordingly to authenticate the user. The Python wrapper part of the component takes an `event_handler` function, and a minimal implementation of this, `handle_event()`, is provided. This stores the token in `SessionState` which is accessible application-wide. The event handler contructs an array of reporting information/data which is passed to the wrapper's `report_event()` function to print output to the console. Change this to suit your needs.

## [3] Flask server hosting API endpoint

The Flask server provides a couple of very simple API endpoints, `/api/ping` (public) and `/api/pong` (protected). They return a timestamped json object. To deal with potential CORS issues, an `@app.after_request` decorator adds the necessary response headers to allow the repsonse to propagate into the Next.js/React component. You can easily write component code to pass an `OnActionRequest` event with `WebRequest` action and auth_kind `'BEARER'` to delegate an authenticated API call to the host, or  code the web request in your Streamlit app.

The Flask server implementation doesn't fully authenticate `/api/pong` and merely checks for the presence of an `Authorization: Bearer <token>` header. (Sorry, didn't want to include a full blown authenticated Flask app with this example. You'll have enough to get going.)

## Starting the application

Whilst all three applications can be started individually in their own directory folders, you can simply enter the `/frontend` folder at the command line and run in separate terminals `yarn <command>`, where `<command>` is one of `dev`, `start-api`, or `start-streamlit`. Ideally, run the commands in this order.

**Note**: When the Streamlit app starts, _the component may not load until it has statically compiled_. Check the Next.js console and when it's compiled, refresh the Streamlit browser window. `yarn build` will pre-compile the Next.js app.

Below is the `scripts` section in `frontend/package.json`. With the `concurrently` package you could start them simultaneously from one command.

```json
"scripts": {
  "dev": "npx next dev -p 3001",
  "build": "next build --debug",
  "start": "npx next start -p 3001",
  "start-streamlit": "streamlit run --server.port 4010 ../app.py",
  "start-api": "python ../server/flask-api.py 8888",
  "typecheck": "tsc"
}
```
