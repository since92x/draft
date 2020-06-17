# Tab路由

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs'

const Tab = createBottomTabNavigator();

function TabApp () {
  return (
    <NavigationContainer>
      <Tab.Navigator>
          <Tab.Screen name="Home" component={HomeScreen} options={{ title: '首页' }} />
          <Tab.Screen name="About" component={AboutScreen} options={{ title: '关于' }} />
      </Tab.Navigator>
    </NavigationContainer>
  )
}
```

# Screen路由

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
const Stack = createStackNavigator();

function ScreenApp () {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} options={{
          title: '首页',
          ...defaultOpts,
        }} />
        <Stack.Screen name="About" component={AboutScreen} options={{
          title: '关于',
          ...defaultOpts,
        }} />
        <Stack.Screen name="WebView" component={WebViewScreen} options={({ route }) => ({
          title: route.params.title,
          ...defaultOpts,
        })} />
      </Stack.Navigator>
    </NavigationContainer>
  )
}
function HomeScreen () {
  return (
    <Button title="About" onPress={() => {
      this.props.navigation.navigate('WebView', {
        url,
        title: '加载中...',
      })
    }} />
  ) 
}
```

# Drawer路由

```javascript
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

function MyDrawer() {
  return (
    <Drawer.Navigator>
      <Drawer.Screen name="地图" component={Map} />
      <Drawer.Screen name="博客" component={Article} />
    </Drawer.Navigator>
  );
}
```
