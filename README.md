# calimoto gpx for rhoenschrat

This code would allow the `kurviger GPX to Garmin GPX Konverter` tool by Rhönschrat to read calimoto GPX files.

Feel free to use as you please.

I just put this together in a few minutes, its mostly meant to show how the GPX file has to be modified.
It works by converting the route to a track and by making a route out of the waypoints. 

```js
function isCalimotoGpx(gpxAsText) {
	return typeof gpxAsText === 'string' 
		&& gpxAsText.includes("calimoto gpx export (https://calimoto.com)");
}

function fixCalimotoGpx(gpxAsText) {
	function convertRouteToTrack(gpx) {
		gpx = gpx.replaceAll("<rte>", `<trk>`);
		gpx = gpx.replace("<rtept ", "<trkseg><rtept ");
		gpx = gpx.replaceAll("<rtept ", `<trkpt `);
		gpx = gpx.replaceAll("</rtept>", `</trkpt>`);
		gpx = gpx.replaceAll("</rte>", `</trkseg></trk>`);
		return gpx;
	}

	function convertWaypointsToRoute(gpx) {
		const tourName = gpxAsText.matchAll(/\<name\>([\w\s]*)\<\/name\>/g).next().value[1];
		gpx = gpx.replace("</metadata>", `</metadata><rte><name>${tourName}</name>`);
		gpx = gpx.replaceAll("<wpt ", "<rtept ");
		gpx = gpx.replaceAll("</wpt>", "</rtept>");
		gpx = gpx.replace("<trk>", `</rte><trk>`);
		return gpx;
	}

	try {
		let gpx = gpxAsText;
		gpx = convertRouteToTrack(gpx);
		gpx = convertWaypointsToRoute(gpx);
		return gpx;
	}
	catch (e) {
		// Return the original if something goes wrong
		console.error(e);
		return gpxAsText;
	}
}
```


## How to use

The code uses the raw string instead of parsed XML. Therfore it has to be run before parsing the GPX:

```js
reader.onload = function(e) {
            let gpxAsText = e.target.result;
            if (isCalimotoGpx(gpxAsText)) {
                gpxAsText = fixCalimotoGpx(gpxAsText);
            }
            const parser = new DOMParser();
            const gpxDoc = parser.parseFromString(gpxAsText, "application/xml");

            // ...
}
```

## Notes

- The code makes a few assumptions about the GPX file, so it could easily break if the calmoto export changes.
- Only works with GPX files exported by the calimoto website. Android and iOS produce different files.
- I made this on my own and in my own free time, not as a calimoto empoyee.
