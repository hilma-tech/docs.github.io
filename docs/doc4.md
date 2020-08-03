---
id: doc4
title: Auth Native
sidebar_label: Auth Native
---

# Auth Native

Auth Native is an autentication package for react-native client applications.

*   [Installation](#installation)
*   [Main exports](#Main-exports)

    > - [AuthProvider](#AuthProvider---component)
    >   > - [AuthProvider Props](#AuthProvider-Props)
    >   > - [Auth Properties](#Auth-Properties)
    >   > - [Accessing the auth object](#Accessing-the-auth-object)
    >    >   > - [via Context](#via-Context)
    >    >   > - [via Refs](#via-Refs)
    > - [createPrivateNavigator](#createPrivateNavigator---function)
    >   > - [PrivateScreen](#PrivateScreen)
    >   > - [MultipleScreen](#MultipleScreen)
    >   > - [HomeScreen](#HomeScreen)
    >   > - [PublicOnlyScreen](#PublicOnlyScreen)

#### Installation

``` bash
$  npm install --save @hilma/auth-native
```

After installing for the first time a file named `variables.js` will be created in the root of your client application.
This file includes all the variables that the package needs to know about, such as the server uri in development and production. the default is `localhost:8080` . You can make any changes as needed.

## Main exports:



### Accessing the auth object:

Now let's see how we actually access all these properties. There are a few ways to access all these propeties:

#### via Context:

`AuthProvider` is mainly built on React's Context API which allows you to access a certain value in any part of an application. There are two ways to access via The Context API:

1. __Recommended:__ Each property has its own context object (kls: `KlsContext` , superAuthFetch: `SuperAuthFetchContext` , and so on...). Each one of these context objects also include a custom hook that returns the property (kls: `useKls` , superAuthFetch: `useSuperAuthFetch` , and so on...).

``` jsx
const superAuthFetch = useSuperAuthFetch();
```

2. __Not Recommended:__ You can access the whole auth object via the `AuthContext` context object. You can also use the `useAuth` hook. __The reason this method is not recommended is because each time you access this context object, you are recieving the whole object. It is very unlikeky you'll need to access the whole abject. The performance of your application will drop significantly. So please use the first method.__

``` jsx
const { superAuthFetch } = useAuth();
```

#### via Refs:

Another way you can access the auth object is via React's refs. This method is useful if you want to access auth outside of components, like in redux middleware or mobx.

``` jsx
import React from 'react';
import { NavigationContainer } from "@react-navigation/native";
import { createStackNavigator } from "@react-navigation/stack";
import { AuthProvider, createAuthRef } from "@hilma/auth-native";

import Admin from './components/Admin';
import Client from './components/Client';

const RootStack = createStackNavigator();

export const authRef = createAuthRef();

const App = () => (
	<NavigationContainer>
		<AuthProvider ref={authRef}>
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

In the file where you use `AuthProvider` import `createAuthRef` , invoke the function. this returns a ref object. A ref object is an object that includes one property which is `current` . `current` holds the value stored in the ref object. In AuthProvider we use forwardRef's to pass the auth object to the provided ref. Now when you want to access the auth object you can export the ref and access the properties via authRef.current:

``` jsx
import { authRef } from './App';

class ErrorStore {
	async addError() {
		const [res, err] = await authRef.current.superAuthFetch('....');
	}
}
```

### createPrivateNavigator - function

This function is based on `createStackNavigator` from `@react-navigation/stack` . If you are not familiar with `react-navigation` , go read the documentation first as this function is completely based on it: https://reactnavigation.org/docs/getting-started.

`createPrivateNavigator` returns a stack just like `createStackNavigator` but it includes 4 more types of screens similar to the `Screen` component. Let's go over them now (all the props of the `Screen` component also apply to these components):

#### PrivateScreen: 

You would use this component if you want your component to only accessible when a user is logged in and authenticated.

##### PrivateScreen Props:

| prop                 | type                     | required | description                                                                                              | default         |
|----------------------|--------------------------|----------|----------------------------------------------------------------------------------------------------------|-----------------|
| componentName             | `string` | false    | if the user is authenticated and its roleAccessConfig includes this string you can access this component | the display name of the component prop |
| redirectComponent  | `React.ComponentType<any>` | false    | if the client is not allowed to access the route then this component will be rendered instead            |                 |
| redirectName | `string` | false    | if the `redirectComponent` is not provided, this prop will be navigated to                             | "Login"         |

#### MultipleScreen: 

If you want components rendered under the same route name but render different components per role. For example: if we have 2 components: `TeacherSchedule` and `StudentSchedule` , now you want them to be rendered under the same route: `Schedule` , You want that when a user with a `Teacher` role is logged in, they will see the `TeacherSchedule` component and for a `Student` role, `StudentSchedule` . `MultipleScreen` was made just for this.

##### MultipleScreen Props: 

| prop                 | type                                     | required | description                                                                                                                           | default |
|----------------------|------------------------------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------|---------|
| components                | `Record<string, React.ComponentType<any>>` | true     | an object where the key being the component name that appears in roleAccessConfig and the value the component that you want to render |         |
| redirectComponent  | `React.ComponentType<any>` | false    | if the client is not allowed to access the route then this component will be rendered instead                                         |         |
| redirectName | `string` | false    | if the `redirectComponent` is not provided, this prop will be navigated to                                                          | "Login" |

#### HomeScreen:

Similar to Multiple screen but instead of going to see if the component names are in the `components` in roleAccessConfig, it goes to `defaultHomePage` . This compnent is good if you want a main screen per role.

##### HomeScreen Props: 

| prop      | type                                     | required | description                                                                                                                           | default |
|-----------|------------------------------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------|---------|
| components     | `Record<string, React.ComponentType<any>>` | true     | an object where the key being the component name that appears in roleAccessConfig and the value the component that you want to render |         |
| redirectComponent | `React.ComponentType<any>` | false    | if the client is not allowed to access the route then this component will be rendered instead                                         |         |
| redirectName | `string` | false | if the `redirectComponent` is not provided, this prop will be navigated to                                                          | "Login" |

#### PublicOnlyScreen:

When a user is authenticated, a `PrivateScreen` will be rendered and if not a regular `Screen` will be rendered.

##### PublicOnlyScreen Props: 

The props are the exact same as `PrivateScreen` .
