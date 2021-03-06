---
id: version-14.4.2-api-scsocket-server
title: SCServerSocket (server, worker)
sidebar_label: SCServerSocket (server, worker)
original_id: api-scsocket-server
---

Note that there are two types of SCSocket objects - One which runs on the client and another which runs on the server.
These two objects have slightly different APIs. This page concerns the server socket

The server-side SCSocket object allows you to interact with a matching client-side socket in real-time.
It also allows you to subscribe and publish to channels. It uses the WebSocket API as the underlying transport.

### Inherits from:

[Emitter](https://github.com/component/emitter)

### Properties:

<table>
    <tr>
        <td>id</td>
        <td>The socket id.</td>
    </tr>
    <tr>
        <td>request</td>
        <td>The Node.js HTTP request which initiated the SCSocket handshake - This object is useful for accessing HTTP headers (including cookies) which relate to the current socket.</td>
    </tr>
    <tr>
        <td>remoteAddress</td>
        <td>
            This socket's client IP address.
        </td>
    </tr>
    <tr>
        <td>exchange</td>
        <td>
            A <a href="/docs/14.4.2/api-exchange">Exchange</a> object which you can use to interact with your SC broker processes.
            You can use it to publish data a channel - Any client which is subscribed to that channel will receive the data.
        </td>
    </tr>
    <tr>
        <td>state</td>
        <td>
            [Only available from v2.0.0 onwards - Otherwise, use the getState() method] The current state of the socket as a string - Can be socket.CONNECTING, socket.OPEN or socket.CLOSED.
        </td>
    </tr>
    <tr>
        <td>
            authState
        </td>
        <td>
            [Only available from v4.0.0 onwards] The authentication state of the socket as a string. Can be either socket.AUTHENTICATED, socket.UNAUTHENTICATED
        </td>
    </tr>
    <tr>
        <td>
            authToken
        </td>
        <td>
            [Only available from v4.0.0 onwards] The auth token (as a plain Object) currently associated with the socket. This property will be null if no token is associated with this socket.
        </td>
    </tr>
    <tr>
        <td>CONNECTING</td>
        <td><b>'connecting'</b> - A string constant which represents this socket's connecting state. See state property.</td>
    </tr>
    <tr>
        <td>OPEN</td>
        <td><b>'open'</b> - A string constant which represents this socket's open/connected state. See state property.</td>
    </tr>
    <tr>
        <td>CLOSED</td>
        <td><b>'closed'</b> - A string constant which represents this socket's closed state. See state property.</td>
    </tr>
    <tr>
        <td>AUTHENTICATED</td>
        <td><b>'authenticated'</b> - A string constant which represents this socket's authenticated authState.</td>
    </tr>
    <tr>
        <td>UNAUTHENTICATED</td>
        <td><b>'unauthenticated'</b> - A string constant which represents this socket's unauthenticated authState.</td>
    </tr>
</table>

### Events:

<table>
    <tr>
        <td>'error'</td>
        <td>
            This gets triggered when an error occurs on this socket. Argument is the error object.
        </td>
    </tr>
    <tr>
        <td>'raw'</td>
        <td>
            This gets triggered whenever the client socket on the other side calls socket.send(...).
        </td>
    </tr>
    <tr>
        <td>'connect'</td>
        <td>
            Triggers when the socket completes the SC handshake phase and is fully connected. The argument passed to the handler is
            the socket connection status object. Note that if you capture socket inside the `scServer.on('connection', handler)` handler,
            then the 'connect' event will already have occurred and therefore won't trigger again. The 'connect' event handler can be attached
            inside the `scServer.on('handshake', handler)` handler instead.
        </td>
    </tr>
    <tr>
        <td>'disconnect'</td>
        <td>
            Happens when the client becomes disconnected from the server. Note that if the socket becomes disconnected during the SC handshake stage,
            then the <code>'connectAbort'</code> event will be triggered instead.
        </td>
    </tr>
    <tr>
        <td>'connectAbort' [since v9.1.0]</td>
        <td>
            Happens when the client disconnects from the server before the SocketCluster handshake has completed (I.e. while <code>socket.state</code> was 'connecting').
            Note that the <code>'connectAbort'</code> event can only be triggered during the socket's handshake phase before the server's <code>'connection'</code> event is triggered.
        </td>
    </tr>
    <tr>
        <td>'close' [since v9.1.1]</td>
        <td>
            Happens when the client disconnects from the server at any stage of the handshake/connection cycle.
            Note that this event is a catch-all for both <code>'disconnect'</code> and <code>'connectAbort'</code> events.
        </td>
    </tr>
    <tr>
        <td>'subscribe'</td>
        <td>Emitted when the matching client socket successfully subscribes to a channel.</td>
    </tr>
    <tr>
        <td>'unsubscribe'</td>
        <td>Occurs whenever the matching client socket unsubscribes from a channel - This includes automatic unsubscriptions triggered by disconnects.</td>
    </tr>
    <tr>
        <td>'badAuthToken'</td>
        <td>
            Emitted when the client tries to authenticate itself with an invalid (or expired) token.
            Note that this event will typically be triggered as part of the socket handshake (before the socket is connected); therefore, it's generally
            better to listen for the <code>'badSocketAuthToken'</code> event directly on the server instance instead.
            The argument passed along with this event is an object with the properties <code>authError</code> and <code>signedAuthToken</code>.
            The authError property holds an error object and the signedAuthToken property holds the auth token which failed the verification step.
        </td>
    </tr>
    <tr>
        <td>'authenticate'</td>
        <td>
            Triggers whenever the client becomes authenticated. The listener will receive the socket's authToken object as argument.
        </td>
    </tr>
    <tr>
        <td>'deauthenticate'</td>
        <td>
            Triggers whenever the client becomes unauthenticated. The listener will receive the socket's old authToken object as argument
            (just before the deauthentication took place).
        </td>
    </tr>
    <tr>
        <td>'authStateChange'</td>
        <td>
            Triggers whenever the socket's <code>authState</code> changes (e.g. transitions between authenticated and unauthenticated states).
        </td>
    </tr>
    <tr>
        <td>'message'</td>
        <td>
            All data that arrives on this socket is emitted through this event as a string.
        </td>
    </tr>
</table>

### Methods:

<table>
    <tr>
        <td>
            getState()
        </td>
        <td>
            Returns the state of the socket as a string constant.
            <ul>
                <li><b>'connecting'</b> - socket.CONNECTING</li>
                <li><b>'open'</b> - socket.OPEN</li>
                <li><b>'closed'</b> - socket.CLOSED</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>disconnect([code, data])</td>
        <td>
            Disconnect this socket client.
            This function accepts two optional arguments:
            A custom error code (a Number - Ideally between 4100 to 4500) and data (which can be either reason message String or an Object).
        </td>
    </tr>
    <tr>
        <td>emit(event, data, [callback])</td>
        <td>
            Emits the specified event on the corresponding server-side socket.
            Note that you cannot use any of the registered SCSocket events (defined above or those specified in engine.io-client).
            This is the same as Socket.io's emit function except that if a callback is provided, your server-side socket will need to respond to this event. <a href="/docs/14.4.2/handling-failure">See example</a>.
            Also note that SC has a default timeout of 10 seconds for responding to events on the other side. You can increase this limit by setting <code>ackTimeout</code> when
            initiating SocketCluster on the server side. See <a href="/docs/14.4.2/api-socketcluster">here</a>.
        </td>
    </tr>
    <tr>
        <td>on(event, handler)</td>
        <td>
            Add a handler for a particular event (those emitted from a corresponding socket on the client). The handler is a function
            in the form: handler(data, res) - The res argument is a function which can be used to send a response to the client socket which
            emitted the event (assuming that the client is expecting a response - I.e. A callback was provided to the emit method).
            The res function is in the form: res(err, message) - To send back an error, you can do either: res('This is an error') or
            res(1234, 'This is the error message for error code 1234').
            To send back a normal non-error response: res(null, 'This is a normal response message').
            Note that both arguments provided to res can be of any JSON-compatible type.
        </td>
    </tr>
    <tr>
        <td>
            off([event, handler])
        </td>
        <td>
            Unbind a previously attached event handler.
        </td>
    </tr>
    <tr>
        <td>
            send(data, [options]);
        </td>
        <td>
              Send some raw data to the client. This will trigger a 'raw' event on the client which will carry the provided data.
        </td>
    </tr>
    <tr>
        <td>getAuthToken()</td>
        <td>Get the decrypted JWT authentication token associated with this socket, if there is one (and it's valid); otherwise, this method will return null (which means that the socket is not authenticated).</td>
    </tr>
    <tr>
        <td>setAuthToken(data, [options])</td>
        <td>
            <p>
                Authenticate the current socket's client by providing it some token data which you can use to validate their identity and/or access rights.
                Generally, you should call this method in response to a successful login attempt by the client (I.e. The client has provided
                a username and password combination which exists in your database).
            </p>
            <p>
                When you call this method, a JWT (token) containing the specified data will be created and attached to both the current server socket and the client socket on the other side of the connection.
                This method can be called multiple times to update the token with new data - Avoid modifying the socket.authToken property directly because otherwise, the change will not propagate to the client.
            </p>
            <p>
                In the simplest case, the data argument to this method would be an object in the form {username: 'bob123'}.
                For a more complex use case, you could also store a list of authorized channel names inside the token (that way, you will be able to check access privileges directly from the socket's token instead of having to read the database every time).
            <p>
                The optional <b>options</b> argument is an Object which can be used to modify the token's behavior - Valid properties include any
                option accepted by the node-jsonwebtoken library's sign method.
                <a href="https://github.com/auth0/node-jsonwebtoken#jwtsignpayload-secretorprivatekey-options-callback">See the list of options here</a>.
                Note that if you use a different algorithm than the default 'HS256', you may need to provide an authPrivateKey and authPublicKey instead of the default authKey.
                See authKey, authPrivateKey and authPublicKey options in SocketCluster constructor <a href="/docs/14.4.2/api-socketcluster">here</a>.
            </p>
            <p>
                Using auth tokens has the following advantages:
                <ul>
                    <li>The user doesn't need to login again if their socket loses its connection - Works well with SC's auto-reconnect feature.</li>
                    <li>
                        An auth token is shared between sockets across multiple tabs in a browser (under the same domain name).
                        If a user logs in on one tab, they will automatically be authenticated on any other tab that they have open.
                        Tokens therefore allow you to attach a user's identity to multiple sockets.
                    </li>
                </ul>
            </p>
        </td>
    </tr>
    <tr>
        <td>deauthenticate()</td>
        <td>
            Disassociate the current socket from its auth token.
            This method will also tell the client socket to remove the token from the browser's cookies.
            This is useful for logging out the user.
        </td>
    </tr>
    <tr>
        <td>kickOut([channel, message, callback])</td>
        <td>
            Forcibly unsubscribe this socket from the specified channel.
            All arguments are optional - If no channel name is provided, it will unsubscribe this socket from ALL channels.
            In addition, the client-side of this socket will emit a 'kickOut' event and a 'dropOut' event to signify that
            it was kicked out of the channel and has dropped out of further messages.
            Note that this doesn't prevent the client from resubscribing to that channel later. You will need to update your middleware
            if you want to achieve this.
        </td>
    </tr>
    <tr>
        <td>
            subscriptions()
        </td>
        <td>
            Returns an array of active channel subscriptions which this socket's client is bound to.
        </td>
    </tr>
    <tr>
        <td>
            isSubscribed(channelName)
        </td>
        <td>
            Check if socket is subscribed to channelName.
        </td>
    </tr>
</table>
