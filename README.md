# Weather App

A small Vue 3 + Vite weather application to fetch and display weather information.

**Features**
- Weather for 5 days (including today) by using the coordinates given.
- App uses `openweathermap` data. Refer following link: `https://openweathermap.org/forecast5` 

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

Open http://localhost:5173 in your browser (Vite default).

Environment / API keys
- App needs `openweathermap` API key, create a `.env` file in the project root and add keys prefixed with `VITE_`, for example:

```text
VITE_WEATHER_API_KEY=your_api_key_here
```

Access environment variables in code via `import.meta.env.VITE_WEATHER_API_KEY`.

Future Improvements
- Accessing given city's cordinates using `https://openweathermap.org/api/geocoding-api` endpoint.
- Add air quality component using `https://openweathermap.org/api/air-pollution` endpoint

Contributing
- Feel free to open issues or submit PRs with improvements.

License & Credits
- Based on the Vue + Vite starter template.

