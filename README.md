# Route Planner - Trucks, Cars, Hikers & Cyclists

A comprehensive single-file HTML route planner with mandatory rest stops, parking locations, and water points for different vehicle types. Supports EU rest regulations for trucks/cars, daily distance planning for hikers and cyclists, and includes Nordic wild camping rules.

## Features

### Truck Support
- EU Regulation EC 561/2006 compliance
- 45-minute breaks every 4.5 hours
- 11-hour daily rest every 9 hours
- 45-hour weekly rest every 6 days
- Axle load restriction checking
- Weight limit verification
- Road restriction warnings

### Car Support
- EU rest regulations (suggested)
- Same rest periods as trucks
- No axle load restrictions
- Parking spot suggestions

### Hiker Support
- Daily segments: 40 km suggested maximum per day
- Rest stops: 11.295 km intervals (main) or 2.275 km (alternative)
- Hiking route search (`route=hiking`, `type=route`)
- Drinking water points with refill locations
- Natural spring locations with safety warnings
- Shelter, hut, and camping area locations
- Hut search near daily stops:
  - Alpine huts (`tourism=alpine_hut`)
  - Wilderness huts (`tourism=wilderness_hut`)
  - Unlocked basic huts (`shelter_type=basic_hut`, `locked=no`)
- Road safety checks - avoids unsafe roads (motorways, trunk roads, primary roads)
- Allowed roads: footway, path, track, steps, bridleway, and roads with `hiking=yes` or `foot=designated`

### Cyclist Support
- Daily segments: 70 km suggested maximum per day
- 4 segments per day, 17 km each
- Segment break markers
- Drinking water points with refill locations
- Natural spring locations with safety warnings
- Hut search near daily stops and segment breaks
- Road safety checks - avoids unsafe roads (motorways, trunk roads, primary roads)
- Allowed roads: cycleway, path, track, secondary, tertiary, unclassified, residential, living_street

### Map Interaction
- Click on map to set start location
- Click on map to set end location
- Click on map to add via points (waypoints)
- Reverse geocoding for clicked locations
- Visual markers for all route points

### Water Points
- Automatic detection of drinking water locations
- Supports `drinking_water:refill=yes`
- Supports `drinking_water=yes`
- Natural spring detection with safety warnings
- "Drink it at your own risk" notice for springs
- Water points shown along entire route for hiking and cycling

### Road Safety
- Automatic route safety checking for hikers and cyclists
- Avoids unsafe road types: motorway, trunk, primary, primary_link, motorway_link, trunk_link
- Safety warnings displayed on map and in sidebar
- Real-time validation of road types along route
- Helps plan safer routes by identifying problematic segments

### Parking & Camping
- Suitable parking spots for trucks/cars
- Nordic wild camping rules by country:
  - **Norway**: 2 nights max, 150m from houses
  - **Sweden**: 1-2 nights, out of sight from houses
  - **Denmark**: WARNING - Designated sites only
  - **Finland**: Short periods, keep distance from homes
- Building proximity filtering (150m minimum)

## Technology Stack

- **OSRM** - Open Source Routing Machine for route calculation
  - Profile: `driving` (trucks/cars), `foot` (hikers), `cycling` (cyclists)
  - API: https://router.project-osrm.org/
- **Nominatim** - Geocoding and reverse geocoding
  - API: https://nominatim.openstreetmap.org/
- **Overpass API** - POI data queries
  - API: https://overpass-api.de/api/interpreter
- **Leaflet.js** - Interactive map display
  - Version: 1.9.4
  - CDN: https://unpkg.com/leaflet@1.9.4/
- **OpenStreetMap** - Map tiles and data

## Usage

### Basic Usage

1. Open `truck-route-planner.html` in a web browser
2. Select vehicle type (Truck, Car, Hiker, or Cyclist)
3. Enter start and destination addresses (or click on map)
4. Set departure time
5. For trucks: Enter total weight, axle load, and number of axles
6. Click "Calculate Route"

### Map Clicking

1. Click one of the mode buttons in the top-right corner:
   - **Start Location** - Click on map to set start
   - **End Location** - Click on map to set end
   - **Via Point** - Click on map to add waypoints
   - **Disable** - Turn off map clicking
2. The address fields will auto-fill with reverse geocoded addresses

### Via Points

- Add multiple waypoints by clicking "Via Point" mode and clicking on the map
- Via points appear in a list with remove buttons
- Routes will pass through all via points in order

## API Rate Limiting

The application implements rate limiting for Nominatim requests:
- 1 second delay between Nominatim API calls
- User-Agent header: `RoutePlanner/1.0`
- Please respect OSM usage policies

## Route Calculation Details

### Truck/Car Routes
- Uses OSRM `driving` profile
- Calculates rest stops based on driving time
- Checks axle load restrictions along route
- Finds parking spots for overnight rests

### Hiker Routes
- Uses OSRM `foot` profile
- Searches for hiking routes (`route=hiking`, `type=route`)
- Daily segments: 40 km maximum suggested
- Rest stops at 11.295 km or 2.275 km intervals
- Water points and shelters along route
- Hut search: alpine huts, wilderness huts, unlocked basic huts
- Road safety validation - warns about unsafe road types
- Allowed roads: footway, path, track, steps, bridleway, or roads with hiking/foot designations

### Cyclist Routes
- Uses OSRM `cycling` profile
- Daily segments: 70 km maximum suggested
- 4 segments per day, 17 km each
- Segment break markers
- Water points along route
- Hut search near daily stops and segment breaks
- Road safety validation - warns about unsafe road types
- Allowed roads: cycleway, path, track, secondary, tertiary, unclassified, residential, living_street

## Marker Colors

### Trucks/Cars
- **Green**: Suitable parking spots
- **Orange**: 45-minute breaks
- **Red**: Daily rest (11 hours)
- **Purple**: Weekly rest (45 hours)
- **Yellow**: Axle load warnings (trucks only)

### Hikers
- **Blue**: Rest at 11.295 km intervals
- **Light Blue**: Alternative rest at 2.275 km intervals
- **Blue**: Drinking water points
- **Green**: Shelters, huts, camping areas (alpine huts, wilderness huts, basic huts)
- **Teal**: Daily segment end (40 km)
- **Yellow**: Safety warnings (unsafe roads)

### Cyclists
- **Teal**: Daily segment end (70 km)
- **Orange**: Segment break (17 km)
- **Blue**: Drinking water points
- **Green**: Huts and shelters near stops
- **Yellow**: Safety warnings (unsafe roads)

## File Structure

```
truck-router/
├── truck-route-planner.html  # Single-file application
└── README.md                 # This file
```

## Browser Compatibility

- **Chrome/Edge**: Full support (recommended)
- **Firefox**: Full support
- **Safari**: Full support
- **Opera**: Requires experimental Web Platform features enabled

## Dependencies

All dependencies are loaded via CDN:
- Leaflet.js 1.9.4 (CSS and JS)
- OpenStreetMap tiles

No build process or package manager required.

## API Services Used

### OSRM Routing
- Endpoint: `https://router.project-osrm.org/route/v1/{profile}/{waypoints}`
- Profiles: `driving`, `foot`, `cycling`
- Free public instance (please use responsibly)

### Nominatim Geocoding
- Endpoint: `https://nominatim.openstreetmap.org/search`
- Reverse: `https://nominatim.openstreetmap.org/reverse`
- Free public instance (rate limited)
- Requires User-Agent header

### Overpass API
- Endpoint: `https://overpass-api.de/api/interpreter`
- Free public instance
- Used for POI queries (parking, water, hiking routes)

## Disclaimer

**Important**: This tool is for planning purposes only. Always verify:
- Local regulations and road restrictions
- Road conditions and accessibility
- Water source safety (especially natural springs)
- Camping permissions and restrictions
- Axle load limits with local authorities

The parking spot suggestions and water point locations are based on OpenStreetMap data and may not be accurate or legal. Always respect private property and local laws.

## License

This project is open source. Please use responsibly and respect the terms of service of the APIs used (OSRM, Nominatim, Overpass).

## Contributing

Contributions are welcome! Please ensure:
- Code follows existing style
- All features work in the single-file format
- API rate limiting is respected
- Error handling is included

## Credits

- **OSRM** - Open Source Routing Machine
- **OpenStreetMap** - Map data and tiles
- **Nominatim** - Geocoding service
- **Overpass API** - POI data queries
- **Leaflet** - Map library

## Support

For issues or questions, please open an issue on GitHub.

---

**Note**: This is a single-file HTML application. No server-side code or build process is required. Simply open the HTML file in a modern web browser.

