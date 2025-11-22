# Weather App

A small Vue 3 + Vite weather application to fetch and display weather information.

**Features**
- Weather forecast for 5 days for a given city.
- App uses `openweathermap` data to get the geo coordinates and the weather data by using following endpoints:
  - http://api.openweathermap.org/geo/1.0/direct?q={city},{state},{country}&limit={limit}&appid={API_key}
  - http://api.openweathermap.org/data/2.5/forecast?lat={lat}&lon={lon}&appid={API_key}

**Requirements**
- Node.js 14+ (Node 18+ recommended)
- npm (or yarn)

**Quick Start**

Install dependencies:

```powershell
npm install
```

Run the dev server:

```powershell
npm run dev
```

Open http://localhost:5173 in your browser.

Environment / API keys
- App needs `openweathermap` API key, create a `.env` file in the project root and add keys prefixed with `VITE_`, for example:

```text
VITE_WEATHER_API_KEY=your_api_key_here
```

Access environment variables in code via `import.meta.env.VITE_WEATHER_API_KEY`.

Future Improvements
- Better UX/UI
- Improve the weather forcast for 5 days UI.

Other APIs that we can utilize

- https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key}


Contributing
- Feel free to open issues or submit PRs with improvements.

License & Credits
- This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
- Weather data and forecasts provided by OpenWeatherMap — https://openweathermap.org/