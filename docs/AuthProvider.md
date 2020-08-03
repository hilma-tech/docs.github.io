---
id: AuthProvider
title: AuthProvider
sidebar_label: AuthProvider
---

`AuthProvider` is the main component needed from this package. It includes all the logic and methods needed for your application. `AuhtProvider` is as wrapping component that uses context to pass all its logic to the rest of your application.

<br />

 Before implemetation, let's examine some of the logic in `AuthProvider` . There's an object stored in the component called `storage` and is of the `Storage` type. This object includes 5 fields, all of which are either a `string` or `null`: 

``` ts
type Storage = Record<'kl' | 'klo' | 'kloo' | 'klk' | 'access_token', null | string>;
```

| key          | description                                           |
|--------------|-------------------------------------------------------|
| kl           | the role key of the authenticated user                |
| klo          | an encoded version of roleAccessConfig in config.json |
| kloo         | a random string                                       |
| klk          | a random string                                       |
| access_token | the current users access token                        |

<br /> 

Most of the methods and properties stored in `AuthProvider` use this object in some way, whether it being for making fetch request, or showing certain components to the client

<br />
<br />

Let's look at an example of how to implement `AuthProvider` :

##### `App.js` :

``` jsx
import React from 'react';
import { NavigationContainer } from "@react-navigation/native";
import { createStackNavigator } from "@react-navigation/stack";
import { AuthProvider } from "@hilma/auth-native";

import Admin from './components/Admin';
import Client from './components/Client';

const RootStack = createStackNavigator();

const App = () => (
	<NavigationContainer>
		<AuthProvider>
			<RootStack.Navigator>
				<RootStack.Screen name="Admin" component={Admin} />
				<RootStack.Screen name="Client" component={Client} />
				<RootStack.Screen name="Login" component={Login} />
			</RootStack.Navigator>
		</AuthProvider>
	</NavigationContainer>
);

export default App;
```

If you want to make this shorter you can import `provide` from the `@hilma/tools` package:

``` jsx
import React from 'react';
import { NavigationContainer } from "@react-navigation/native";
import { createStackNavigator } from "@react-navigation/stack";
import { AuthProvider } from "@hilma/auth-native";
import { provide } from '@hilma/tools';

import Admin from './components/Admin';
import Client from './components/Client';

const RootStack = createStackNavigator();

const App = () => (
	<RootStack.Navigator>
		<RootStack.Screen name="Admin" component={Admin} />
		<RootStack.Screen name="Client" component={Client} />
		<RootStack.Screen name="Login" component={Login} />
	</RootStack.Navigator>
);

export default provide(NavigationContainer, AuthProvider)(App);
```

<br />
<br />

### AuthProvider Props


```ts
interface AuthProviderProps {
    /**
     * If true, when a fetch request is made via superAuthFetch and the user is unauthorized, the logout function will be called
     * Defaults to false
     */
	logoutOnUnauthorized?: boolean;
    /**
     * When `AuthProvider` mounts it doesn't render its children until it fetchs from AsyncStorage, instead it will render the fallback
     * Defaults to `<AppLoading />` from `expo`
     */
	fallback?: JSX.Element;
	/**
     * If you want a reference to the auth object pass a ref
     */
	ref?: React.Ref<HilmaAuth>;
}
```

<br />
<br />

### Auth Properties

Now let's look at whats included in the auth object. The object implements the `HilmaAuth` interface:

```ts
interface HilmaAuth {
    /**
     * Object including `klo` and `kl` from `storage` 
     */
    kls: {
        kl: string | null;
        klo: string | null;
    };
    /**
     * Sets the key value pair in `storage` and AsyncStorage
     * @param key The item key you want to store
     * @param value The value
     */
    setAuthItem(key: keyof Storage, value: string): Promise<void>;
    /**
     * Returns the auth storage item
     * @param key The key
     */
    getAuthItem(key: keyof Storage): string | null;
    /**
     * Removes the item from `storage` and AsyncStorage
     * @param key The item key you want to remove
     */
    removeAuthItem(key: keyof Storage): Promise<void>;
    /**
     * Returns the acces token
     */
    getAccessToken(): string | null;
    /**
     * If true, the user is authenticated
     */
    isAuthenticated: boolean;
    /**
     * Removes all storage in `storage` 
     */
    logout(): Promise<void>;
    /**
     * Will fetch from data from the server
     * @param input the url.the apiUrl in variables.js will be used to make the request
     * @param init 
     * @returns A Promise with the resloved value being a tuple of the response and the errror (if one accured). the response will already be in json format
     */
    superAuthFetch(input: RequestInfo, init?: RequestInit | undefined): Promise<[any, any]>;
    /**
     * Log in function
     * @param body The body you want to send to the server. example: { email: `hilma@gmail.com`, password: `somepassword` }
     * @param input The login endpoint. Defaults to `/api/CustomUsers/elogin`
     * @returns A Promise with the resolved value being an object: `{ success: boolean; user?: any; msg?: any; }`
     */
    login(body: Record<'password' | 'username' | 'email', string>, input?: RequestInfo): Promise<{ success: boolean; user?: any; msg?: any; }>;
}
```