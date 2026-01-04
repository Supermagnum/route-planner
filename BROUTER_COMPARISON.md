# BRouter Implementation Comparison

This document compares how BRouter handles breaks, water points, and huts/cabins versus our current implementation.

## Rest Stops / Breaks

### BRouter Approach
- **Hiking**: Every 11.295 km (old Norwegian mile) or 2.275 km (1/4 mile) alternative
- **Maximum daily distance**: 40 km per day
- **Cycling**: Main rest every 28.24 km, alternative rest every 5.69 km
- **Maximum daily distance**: 100 km per day for trekking cyclists
- Rest stops are automatically added as waypoints in GPX output

### Our Implementation
- **Hiking**: Every 11.295 km (main) and 2.275 km (alternative) ✅ **MATCHES**
- **Maximum daily distance**: 40 km per day ✅ **MATCHES**
- **Cycling**: Segment breaks every 17 km, daily segments 70 km per day
- Rest stops are calculated and displayed on map

**Status**: ✅ Rest stop distances match BRouter. Our cycling implementation differs (we use 17 km segments vs BRouter's 28.24 km main rest).

---

## Water Points

### BRouter Approach
- **Search radius**: 2 km around rest stops
- **OSM tags searched**:
  - `amenity=drinking_water`
  - `amenity=fountain`
  - `natural=spring`
- **Distance filtering**: Minimum 4 km between water points
- **Spring warnings**: Includes warnings for natural springs ("Drink at your own risk")
- **POI integration**: Water points are included in waypoint names with distances (e.g., "Rest Stop 1 (11.3 km) | Water: 450m")

### Our Implementation
- **Search radius**: 2 km around rest stops ✅ **MATCHES**
- **OSM tags searched**:
  - `drinking_water:refill=yes` ❌ **DIFFERENT**
  - `drinking_water=yes` ✅ **MATCHES** (but should be `amenity=drinking_water`)
  - `natural=spring` ✅ **MATCHES**
  - Missing: `amenity=fountain` ❌ **MISSING**
- **Distance filtering**: Minimum 3 km between water points ❌ **DIFFERENT** (BRouter uses 4 km)
- **Spring warnings**: ✅ **MATCHES** ("Drink at your own risk")

**Status**: ⚠️ **NEEDS UPDATE**
- Should search for `amenity=drinking_water` and `amenity=fountain` instead of/in addition to `drinking_water:refill=yes`
- Should use 4 km minimum distance (currently 3 km)

---

## Huts / Cabins / Shelters

### BRouter Approach
- **Search radius**: 5 km around rest stops
- **OSM tags searched**:
  - `tourism=alpine_hut`
  - `tourism=wilderness_hut`
  - `tourism=hut`
  - `tourism=cabin`
- **Priority order**: alpine_hut → wilderness_hut → hut → cabin
- **Accessibility filtering**:
  - Only includes cabins with `access=yes` OR no access restriction
  - Only includes cabins with `locked=no` OR no locked tag
  - Excludes network cabins requiring membership (e.g., `dnt:lock=yes`)
- **Network information**: Includes network information in waypoint names when available
- **POI integration**: Cabins/huts are included in waypoint names with distances (e.g., "Main Rest 1 (28.24 km) | Cabin: 1200m")

### Our Implementation
- **Search radius**: 3 km around rest stops ❌ **DIFFERENT** (BRouter uses 5 km)
- **OSM tags searched**:
  - `tourism=alpine_hut` ✅ **MATCHES**
  - `tourism=wilderness_hut` ✅ **MATCHES**
  - `amenity=shelter` with `shelter_type=basic_hut` and `locked=no` ⚠️ **DIFFERENT** (BRouter uses `tourism=hut` and `tourism=cabin`)
  - `amenity=shelter` ✅ **MATCHES** (but not in BRouter's list)
  - `tourism=camp_site` ⚠️ **NOT IN BROUTER**
  - `leisure=picnic_table` ⚠️ **NOT IN BROUTER**
  - Missing: `tourism=hut` ❌ **MISSING**
  - Missing: `tourism=cabin` ❌ **MISSING**
- **Priority order**: alpine_hut → wilderness_hut → basic_hut → shelter → camp_site → picnic_table
- **Accessibility filtering**:
  - Only checks `locked=no` for basic_hut ❌ **INCOMPLETE**
  - Missing: `access=yes` check ❌ **MISSING**
  - Missing: Network membership exclusion (e.g., `dnt:lock=yes`) ❌ **MISSING**

**Status**: ⚠️ **NEEDS UPDATE**
- Should increase search radius to 5 km (currently 3 km)
- Should add `tourism=hut` and `tourism=cabin` to search
- Should implement proper accessibility filtering (`access=yes` or no restriction, `locked=no` or no locked tag)
- Should exclude network cabins requiring membership (`dnt:lock=yes`)
- Should include network information in waypoint names

---

## Summary of Required Changes

1. **Water Points**:
   - Add `amenity=fountain` to search
   - Change `drinking_water=yes` to `amenity=drinking_water`
   - Increase minimum distance from 3 km to 4 km

2. **Huts/Cabins**:
   - Increase search radius from 3 km to 5 km
   - Add `tourism=hut` and `tourism=cabin` to search
   - Implement proper accessibility filtering:
     - `access=yes` OR no access restriction
     - `locked=no` OR no locked tag
   - Exclude network cabins with `dnt:lock=yes`
   - Include network information in waypoint names

3. **Optional Enhancements**:
   - Include POI distances in waypoint names (e.g., "Rest Stop 1 (11.3 km) | Water: 450m")
   - Add network information to cabin waypoint names

---

## References

- BRouter GitHub: https://github.com/Supermagnum/brouter/tree/master
- BRouter Documentation: See README.md in BRouter repository

