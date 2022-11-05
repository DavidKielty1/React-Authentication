React Authentication.

Authentication - needed if content is to be protected, not accessible by everyone.

May want to close down API end-points to where we send requests within the react app, not just url navigation.

Authentication is a 2 step process.
1 . Get access/permission _provide credentials(logging in)_ create an account -> data is sent to server to verify details are correct password, email.
2 . Send request to protected resource. _use permission to send requests to other API endpoints_

Client sends request to server (external or internal API); server grants requests or denies request. Permission can be used to deny/grant.

_Permission_ is not as simple as server responding 'yes/no'
. server-side sessions
. authenitation tokens

_Server-side_ - once granted access, server stores a unit identifier for every user.
. unique identifier is stored on server, also sent back to client.
. client sents identifier along with requests

disadvantage - requires tightly coupled front-end and back-end APIs (on same server).
. server should be 'stateless', should not store user unique identifiers.

_Authentication token_ - allows decoupling of back-end and front-end
. send credentials, server validates credentials by comparing email, password to what is stored in database.
. server creates a server token - algorithm encodes credentials into one string with encryption.
. key is sent to hash data into string which is only known by server.
. React app attaches token with key to future requests to protected resources on the server.
. server can tell if the token/key was created by the server.

. Tokens created in JSON Web Token (JWT)

_fetching POST signUp with email & password firebase_
. require firebase specific API for signing up

_fetching POST logIn with email & password firebase_
. require firebase specific API endpoint URL for logging in

_storing authentication token_
. authentication state (authenticated(logged in)/unauthenticated (!logged in)) needed for many other components throughout app
. app-wide state => ContextAPI/redux.
. using contextAPI to remove need to install extra 3rd party package. - also not a state which will change frequently so no performance issues.

_using ContextAPI_
. separate file in src => store folder. auth-context.js
. import React from 'react' => _React.createContext();_
. AuthContextProvider value={contextValue}
. . contextValue holds token, isLoggedIn state, as well as pointers to loginHandler and logoutHandler
. . . Handlers merely setToken to null (logout), or token to token string received in response from API (token)

AuthContextProvider will be exported as a named "export const AuthContextProvider".
. AuthContextProvider will wrap around the entire app in the index.js component.

Import { _useContext_ } from 'react' in the components that require the contextValue.
. i.e. in the AuthForm component within this app.
. const authCtx = useContext(AuthContext);

Import { _useRef_ } from 'react' to store form inputs into constants.
. add ref={_constant which holds ref value_} to input element in form
. const newPasswordInputRef = useRef();
. const submitHandler = (e) => {
. const enteredNewPassword = newPasswordInputRef.current.value;

Logout context function -
. Firebase does not store token on server side, we merely need to set token to null
. authCtx.logout(); -> logoutHandler = () => { setToken(null) }

_Navigation guards_ Protecting pages in front end app.
. Employed within the app route

_Local Storage stayLoggedIn approach_
. logIn Firebase API has expiry rate
. return object has idToken and expiresIn keys. Default 1hr. (Advanced: refresh with refresh token)
. in order to stay logged in the token must be stored somewhere outside of the react state.
. . if stored in a JS variable, the value would be cleared when page reloads.
. browser storage mechanisms = cookies
. . easier option -> local storage (vulnerable to cross-site scripting attacks).

. login stores token not just in state, but also in local storage. when logged out we need to clear it in local storage.
. in addition, when page loads we want to look in to storage to find token and use it without user needing to send new request first
. new Date().getTime(); token.expiresIn.toISOString() -> set number to string

_Remove and add items to local storage_
. localStorage.setItem({'token', token})
. localStorage.removeItem('token')

_Caluculating token expire times and auto-logging out_
. check auth-context.js for timing caluculations code

_Clearing logout timer if user logs out manually_
. clearTimeout(logoutTimer) within logoutHandler.
. logoutTimer = setTimeout(logoutHandler, remainingTime)

_useCallback_
. setting logoutHandler as a dependency in useEffect (autho-context.js) to protect from infinite loops or unncecessary side-effects
. useCallback requires no dependencies in this case as localStorage.removeItem and clearTimeout are browser functions not specific to current react app.
. setToken() is part of useState() thus react ensures this will never change (triggering dependency useEffect)
. logoutTimer is outwith the component as a global varaiable so will not be reloaded triggering useEffect either.
. . thus these are not needed to be added as dependencies to useCallback();
