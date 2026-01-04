# Route Planner - Trucks, Cars, Hikers & Cyclists

A comprehensive single-file HTML route planner with mandatory rest stops, parking locations, and water points for different vehicle types. Supports EU rest regulations for trucks/cars, daily distance planning for hikers and cyclists, and includes Nordic wild camping rules.

## Features

### Truck Support
- EU Regulation EC 561/2006 compliance
- 45-minute breaks every 4.5 hours (±20 minutes tolerance)
- 11-hour daily rest every 9 hours (±20 minutes tolerance)
- 45-hour weekly rest every 6 days
- **Optimized calculation**: Uses route distance/time directly instead of iterating all coordinate points
- Axle load restriction checking
- Weight limit verification
- Road restriction warnings

### Car Support
- EU rest regulations (suggested)
- Same rest periods as trucks (with ±20 minutes tolerance)
- **Optimized calculation**: Uses route distance/time directly instead of iterating all coordinate points
- No axle load restrictions
- Parking spot suggestions

### Hiker Support
- Daily segments: 40 km suggested maximum per day
- Rest stops: 11.295 km intervals (main) or 2.275 km (alternative)
- Hiking route search (`route=hiking`, `type=route`)
- Drinking water points with refill locations
- Natural spring locations with safety warnings
- Shelter, hut, and camping area locations (matches BRouter specification)
- Hut search near daily stops (5 km radius, matches BRouter):
  - Alpine huts (`tourism=alpine_hut`) - Priority 1
  - Wilderness huts (`tourism=wilderness_hut`) - Priority 2
  - Huts (`tourism=hut`) - Priority 3
  - Cabins (`tourism=cabin`) - Priority 4
  - Shelters (`amenity=shelter`) - Priority 5
  - Camp sites (`tourism=camp_site`) - Priority 6
  - Picnic tables (`leisure=picnic_table`) - Priority 7
- **Accessibility filtering**: Only includes accessible and unlocked spots
  - `access=yes` OR no access restriction
  - `locked=no` OR no locked tag
  - Excludes network cabins requiring membership (e.g., `dnt:lock=yes`)
- **Network information**: Includes network information (e.g., DNT) in waypoint names when available
- **Distance filtering**: Minimum 1300 m between huts/shelters to prevent map clutter
- Road safety checks - avoids unsafe roads (motorways, trunk roads, primary roads)
- **Allowed roads (priority order, highest to lowest)**:
  1. `highway=path`
  2. `highway=footway`
  3. `highway=pedestrian`
  4. `highway=bridleway`
  5. `highway=steps`
  6. `highway=living_street`
  7. `highway=track`
  8. `highway=service`
- Routes automatically prioritize higher-priority roads when available

### Cyclist Support
- Daily segments: 70 km suggested maximum per day
- 4 segments per day, 17 km each
- Segment break markers
- Drinking water points with refill locations
- Natural spring locations with safety warnings
- Hut search near daily stops and segment breaks (5 km radius, matches BRouter)
- **Priority order**: alpine_hut → wilderness_hut → hut → cabin → shelter → camp_site → picnic_table
- **Accessibility filtering**: Only accessible and unlocked spots (access=yes or no restriction, locked=no or no locked tag)
- **Distance filtering**: Minimum 1300 m between huts/shelters to prevent map clutter
- Road safety checks - avoids unsafe roads (motorways, trunk roads, primary roads)
- Allowed roads: cycleway, path, track, secondary, tertiary, unclassified, residential, living_street

### Map Interaction
- Click on map to set start location
- Click on map to set end location
- Click on map to add via points (waypoints)
- **Multiple via points support**: Add unlimited via points, all are included in route calculation
- Via points are processed in order: start → via1 → via2 → ... → viaN → destination
- Remove individual via points from the list
- Reverse geocoding for clicked locations
- Visual markers for all route points
- **Calculation cancellation**: If you change inputs during calculation, the old calculation is automatically cancelled and a new one starts

### Water Points
- Automatic detection of drinking water locations (matches BRouter specification)
- Supports `amenity=drinking_water`
- Supports `amenity=fountain`
- Natural spring detection with safety warnings (`natural=spring`)
- "Drink it at your own risk" notice for springs
- Water points shown along entire route for hiking and cycling
- **Search radius**: 2 km around rest stops (matches BRouter)
- **Distance filtering**: Minimum 4 km between water points to prevent map clutter (matches BRouter)

### Road Safety
- Automatic route safety checking for hikers and cyclists
- **BRouter integration** (recommended for hikers/cyclists): When endpoint is configured, uses BRouter's routing engine with built-in safety rules
  - **Superior routing**: BRouter has advanced routing profiles specifically designed for hiking and cycling
  - **Built-in safety rules**: Automatically avoids forbidden highways and unsafe roads at routing level
  - **Elevation awareness**: Considers elevation data for more accurate routing
  - **Profile-based routing**: Uses `hiking-mountain` profile for hikers and `trekking` profile for cyclists
  - **No post-processing needed**: Routes are calculated with safety rules built-in
  - Configure endpoint: `http://localhost:17777/brouter` (local) or `http://brouter.de/brouter` (public)
- **OpenRouteService integration** (optional fallback): When API key is provided, automatically excludes unsafe roads at routing level
  - Native exclusion of highways, tollways for hikers/cyclists
  - Used as fallback if BRouter is not configured
- **OSRM route optimization** (default fallback): Automatically selects safest route from OSRM alternatives
  - Compares multiple route options and picks the one with fewest unsafe segments
  - Adds intermediate waypoints to avoid problematic areas when possible
  - Used as final fallback if BRouter and OpenRouteService are not available
- Avoids unsafe road types: motorway, trunk, primary, primary_link, motorway_link, trunk_link
- Safety warnings displayed on map and in sidebar for any remaining unsafe segments
- Real-time validation of road types along route
- Helps plan safer routes by identifying and avoiding problematic segments

### Parking & Camping
- **Parking spots for trucks/cars**: Only at intersections and rest areas
  - Rest areas (highway=rest_area or amenity=rest_area)
  - Intersections (nodes where 2+ highway ways meet)
  - Building proximity filtering (150m minimum for intersections)
- **Country border respect**: If route stays within one country (start, end, and via points in same country), parking spots and rest locations are filtered to only show locations within that country to avoid customs issues
- Nordic wild camping rules by country:
  - **Norway**: 2 nights max, 150m from houses
  - **Sweden**: 1-2 nights, out of sight from houses
  - **Denmark**: WARNING - Designated sites only
  - **Finland**: Short periods, keep distance from homes

## Technology Stack

### Routing Services

- **BRouter** (recommended for hikers/cyclists, optional)
  - **Best routing engine** for hiking and cycling with built-in safety rules
  - Elevation-aware routing with customizable profiles
  - Automatically avoids forbidden highways and unsafe roads at routing level
  - Profiles: `hiking-mountain` (hikers), `trekking` (cyclists), `car-fast` (cars), `car-eco` (trucks)
  - **Setup options**:
    - **Local instance**: Run BRouter server on `http://localhost:17777/brouter`
      - Download from: https://github.com/abrensch/brouter
      - Requires Java and routing data segments (download from brouter.de/brouter/segments4/)
    - **Public instance**: Use `http://brouter.de/brouter` (if available)
  - API format: `/brouter?lonlats=lon1,lat1|lon2,lat2&profile=...&format=geojson`
  - **Priority**: Used first for hikers/cyclists if endpoint is configured
  - Falls back to OpenRouteService or OSRM if not configured or if request fails

- **OpenRouteService** - Alternative routing service (optional fallback, requires free API key)
  - Profile: `driving-car` (trucks/cars), `foot-walking` (hikers), `cycling-regular` (cyclists)
  - API: https://api.openrouteservice.org/v2/directions/
  - **Native road exclusion**: Automatically excludes unsafe roads (motorways, trunk, primary) for hikers/cyclists
  - Free tier: 2000 requests/day
  - Registration: https://openrouteservice.org/dev/#/signup (no credit card required)
  - **Priority**: Used as fallback if BRouter is not configured
  - Falls back to OSRM if API key is not provided or if request fails

- **OSRM** - Open Source Routing Machine (default fallback, free, no API key required)
  - Profile: `driving` (trucks/cars), `foot` (hikers), `cycling` (cyclists)
  - API: https://router.project-osrm.org/
  - **Priority**: Used as final fallback if BRouter and OpenRouteService are not available
  - Includes route optimization for hikers/cyclists to avoid unsafe roads

### Other Services

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

1. Open `index.html` in a web browser
2. Select vehicle type (Truck, Car, Hiker, or Cyclist)
3. **Configure routing service** (recommended for hikers/cyclists):
   - **BRouter** (best): Enter BRouter endpoint (e.g., `http://localhost:17777/brouter`)
     - Provides superior routing with built-in safety rules
     - Elevation-aware routing
   - **OpenRouteService** (fallback): Enter API key (get free key at openrouteservice.org)
     - 2000 requests/day free tier
     - Key is saved in browser for convenience
4. Enter start and destination addresses (or click on map)
5. Set departure time
6. For trucks: Enter total weight, axle load, and number of axles
7. (Optional) Add via points by clicking "Via Point" button and clicking on the map
8. Click "Calculate Route"
9. **Note**: You can change inputs (addresses, via points, etc.) during calculation - the old calculation will be automatically cancelled and a new one will start

### Map Clicking

1. Click one of the mode buttons in the top-right corner:
   - **Start Location** - Click on map to set start
   - **End Location** - Click on map to set end
   - **Via Point** - Click on map to add waypoints
   - **Disable** - Turn off map clicking
2. The address fields will auto-fill with reverse geocoded addresses

### Via Points

- **Unlimited via points**: Add as many waypoints as needed
- Add via points by clicking "Via Point" mode and clicking on the map
- Via points appear in a list with remove buttons
- Routes will pass through all via points in the order they were added: start → via1 → via2 → ... → viaN → destination
- Via points are preserved when routes are recalculated
- When route optimization adds avoidance waypoints, original via point order is maintained

### GPX Export

- Export calculated routes to GPX format for use in GPS devices and mapping software
- Includes complete route track with all coordinates
- Waypoints: start, destination, via points, rest stops, safety warnings
- Metadata: route distance, duration, vehicle type, creation timestamp
- File naming: `route_{vehicleType}_{date}.gpx`
- Compatible with Garmin devices, navigation apps, and mapping software

## API Rate Limiting

The application implements rate limiting for Nominatim requests:
- 1 second delay between Nominatim API calls
- User-Agent header: `RoutePlanner/1.0`
- Please respect OSM usage policies

## Route Calculation Details

### Truck/Car Routes
- Uses OSRM `driving` profile
- **Optimized rest stop calculation**: 
  - Uses route distance/time directly (no iteration through all coordinate points)
  - Calculates rest stops based on cumulative distance/time thresholds
  - ±20 minutes tolerance for breaks and daily rest
  - Significant performance improvement for long routes
- Checks axle load restrictions along route
- Finds parking spots for overnight rests (intersections and rest areas only)
- **Country border filtering**: If route stays within one country, all parking spots and rest locations are filtered to that country

### Hiker Routes
- **Routing**: 
  - OpenRouteService `foot-walking` profile (if API key provided) - automatically excludes unsafe roads
  - OSRM `foot` profile (default) - with route optimization to avoid unsafe roads
- Searches for hiking routes (`route=hiking`, `type=route`)
- Daily segments: 40 km maximum suggested
- Rest stops at 11.295 km or 2.275 km intervals
- Water points and shelters along route
- **Water point filtering**: Minimum 4 km between water points
- **Hut filtering**: Minimum 1300 m between huts/shelters
- **Country border respect**: If route stays within one country, water points and shelters are filtered to that country
- **Hut search** (5 km radius, matches BRouter): alpine huts, wilderness huts, huts, cabins, shelters, camp sites, picnic tables
- **Accessibility**: Only accessible and unlocked spots (access=yes or no restriction, locked=no or no locked tag)
- Road safety validation - automatically selects safest route from alternatives
- Allowed roads: footway, path, track, steps, bridleway, or roads with hiking/foot designations

### Cyclist Routes
- **Routing**:
  - OpenRouteService `cycling-regular` profile (if API key provided) - automatically excludes unsafe roads
  - OSRM `cycling` profile (default) - with route optimization to avoid unsafe roads
- Daily segments: 70 km maximum suggested
- 4 segments per day, 17 km each
- Segment break markers
- Water points along route
- **Water point filtering**: Minimum 4 km between water points
- **Hut filtering**: Minimum 1300 m between huts/shelters
- **Country border respect**: If route stays within one country, water points and shelters are filtered to that country
- Hut search near daily stops and segment breaks
- Road safety validation - automatically selects safest route from alternatives
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

## Technical Features

### Route Calculation
- **Multiple routing services**: OSRM (default) and OpenRouteService (optional)
- **Multiple via points**: Supports unlimited via points in route calculation
- **Calculation cancellation**: Automatically cancels old calculations when inputs change
- **Route optimization**: For hikers/cyclists, automatically finds safest routes avoiding unsafe roads
- **Optimized rest stop calculation**: For trucks/cars, uses route distance/time directly instead of iterating all coordinate points (significant performance improvement)
- **Country border respect**: Automatically detects if route stays within one country and filters rest stops, parking spots, water points, and shelters to that country to avoid customs issues
- **Error handling**: Graceful error handling with user-friendly messages

### State Management
- **Calculation tracking**: Prevents race conditions when multiple calculations are triggered
- **Via point management**: Maintains order and allows individual removal
- **UI state**: Loading states, button disabling, and progress indicators

## File Structure

```
truck-router/
├── index.html  # Single-file application
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

### OSRM Routing (Default)
- Endpoint: `https://router.project-osrm.org/route/v1/{profile}/{waypoints}`
- Profiles: `driving`, `foot`, `cycling`
- Free public instance (please use responsibly)
- Used for all vehicle types by default
- Includes route alternatives and safety optimization for hikers/cyclists

### OpenRouteService (Optional)
- Endpoint: `https://api.openrouteservice.org/v2/directions/{profile}`
- Profiles: `driving-car`, `foot-walking`, `cycling-regular`
- Requires free API key (2000 requests/day)
- Registration: https://openrouteservice.org/dev/#/signup
- Automatically used for hiker/cyclist routes when API key is provided
- Native road exclusion: Excludes highways and tollways at routing level
- Falls back to OSRM if API key not provided or request fails

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
- **OpenRouteService** - Alternative routing service with road exclusions
- **OpenStreetMap** - Map data and tiles
- **Nominatim** - Geocoding service
- **Overpass API** - POI data queries
- **Leaflet** - Map library

## Support

For issues or questions, please open an issue on GitHub.

---

**Note**: This is a single-file HTML application. No server-side code or build process is required. Simply open the HTML file in a modern web browser.

