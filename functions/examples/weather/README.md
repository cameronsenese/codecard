### Node.js Weather Fn

Create a new Fn weather function by typing:

	fn init --runtime node --trigger http weather
	cd weather

Modify the file [func.js](func.js) to look like this:
```
const fdk=require('@fnproject/fdk');
const axios = require('axios');

fdk.handle(function(input, ctx) {
  let url = 'http://api.openweathermap.org/data/2.5/weather?appid=[APP_ID]&q=San%20Francisco'
  var codeCardData = getWeather(url)
  return codeCardData
})

const getWeather = async url => {
  try {
    const response = await axios.get(url)
    let jsonData = response.data
    var codeCardJson = {
      template: 'template1',
      title: jsonData.name + ' Weather',
      subtitle: jsonData.main.temp + ' - ' +  jsonData.weather[0].description,
      bodytext: 'More weather info...',
      icon: jsonData.weather[0].icon,
      backgroundColor: 'white'
    }
    return codeCardJson
  } catch (error) {
    console.log('ERROR:',error)
    return null
  }
};

```

Get a free [openweather API key](https://openweathermap.org/appid), and replace the url variable [APP_ID] with your new key.

Modify the file [package.json](package.json) to include the axios http library as a dependency.

```
...
	"dependencies": {
     	"@fnproject/fdk": "0.x",
         "axios": "^0.18.0"
...
```

Deploy your function:
	
	fn deploy --app codecard --local

and test:

	http://linux-instance-public-ip:8080/t/codecard/weather-trigger
Now you can use the [Code Card Terminal](https://github.com/cameronsenese/codecard/tree/master/terminal) to assign the Fn function to a button.
