---
id: AccessingAuth
title: Accessing the auth object
sidebar_label: Accessing the auth object
---

Now let's see how we actually access all the properties from the previous chapter. There are a few ways to access all these propeties:

### via Context

`AuthProvider` is mainly built on React's Context API which allows you to access a certain value in any part of an application. There are two ways to access via The Context API:

1. __Recommended:__ Each property has its own context object (kls: `KlsContext` , superAuthFetch: `SuperAuthFetchContext` , and so on...). Each one of these context objects also include a custom hook that returns the property (kls: `useKls` , superAuthFetch: `useSuperAuthFetch` , and so on...).

``` jsx
const superAuthFetch = useSuperAuthFetch();
```

2. __Not Recommended:__ You can access the whole auth object via the `AuthContext` context object. You can also use the `useAuth` hook. __The reason this method is not recommended is because each time you access this context object, you are recieving the whole object. It is very unlikeky you'll need to access the whole abject. The performance of your application will drop significantly. So please use the first method.__

``` jsx
const { superAuthFetch } = useAuth();
```

### via Refs

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