# SVG → OpenSCAD polygon() (local, drag-&-drop)

Convert SVG paths into an OpenSCAD polygon(points, paths) with zero installs. This single-file HTML app runs fully offline in your browser, supports cubic Bézier curves, merges duplicate vertices, with copy-to-clipboard and Download .scad.

⸻

Adapted from https://gist.github.com/phrz/67fad11a9a3cd4f18f56e1e3fdcf302f

⸻

Why this small tool?

Most workflows either (a) import SVG directly in OpenSCAD, which doesn’t expose the underlying points for editing; or (b) rely on Inkscape extensions or scripts that require installation and may generate heavyweight or version-sensitive code. This tool aims for a third path:
	•	Local, no install: open the HTML file, drag an SVG, done.
	•	Explicit geometry: you get the points array and paths indices for precise editing.
	•	Curve control: adjustable Bézier sampling (by segments), no duplicate vertex at t=0.
	•	Practical toggles: Invert Y (SVG↔OpenSCAD axis), Force close open subpaths, decimal rounding.
	•	Convenience: live SVG preview, Copy code, Download .scad.

Related tools & how this differs

Tool	Install?	Points list visible?	Bézier support	Notes
This app (local HTML)	No	Yes (points + paths)	C / S (cubic) via sampling	Drag-&-drop, offline, copy/download.
OpenSCAD import(\"…svg\")	No	No (geometry only)	Tessellated by import settings	Great for quick import, but no access to point data.  ￼
Inkscape “Paths to OpenSCAD”	Yes (Inkscape + ext.)	Often modules w/ polygons	Varies by fork	Mature but extension-based, sometimes unmaintained.  ￼ ￼ ￼
Inkscape “SVG→OpenSCAD Bézier” ext.	Yes (Inkscape + ext.)	Yes	Full Bézier support (plugin)	Good option if you live in Inkscape.  ￼ ￼
SVG2SCAD scripts	Yes (Python/Node)	Yes	Varies	Script-driven pipeline alternative.  ￼ ￼

Note: OpenSCAD can import SVG as a polygonal shape, but does not expose the point list you could iterate over in code. If you need those points, you need a converter.  ￼

⸻

What it supports
	•	SVG <path> commands: M, L, Z, C (matching your Python script), plus H, V, S for convenience (absolute & relative).
	•	Cubic Bézier sampling with user-set segments per curve (uniform in t).
	•	No duplicate vertex at the start of curves (we skip t=0 on purpose).
	•	Decimal rounding & duplicate-vertex merging (post-rounding).
	•	Force close open subpaths (optional).
	•	Invert Y (SVG’s Y-down → OpenSCAD’s Y-up).

Known limitations
	•	Only <path> elements are parsed; convert shapes to paths first.
	•	Group/path transforms are ignored—apply transforms before export.
	•	Quadratic Béziers (Q/T) and arcs (A) are not currently parsed (same scope as the original Python).

⸻

Quick start
	1.	Clone the repository or juste save the index.html and open it in your browser.
	2.	Prepare your SVG (see tips below), then drag & drop it onto the page.
	3.	Adjust:
	•	Decimals (rounding),
	•	Bezier segments (curve resolution),
	•	Invert Y (if your model is flipped vertically),
	•	Force close (ensure closed loops).
	4.	Click Copy code or Download .scad.

You’ll get something like:

polygon(
  points=[
    [x0,y0],
    [x1,y1],
    ...
  ],
  paths=[
    [0,1,2,3,0],   // closed contour
    [4,5,6,7,4]    // another subpath (e.g., a hole)
  ],
  convexity=10
);


⸻

SVG prep tips (best results)
	•	Flatten to paths: in Inkscape, select all → Path → Object to Path. Remove strokes/markers effects that won’t convert to geometry.  ￼
	•	Apply transforms: ensure the coordinates are final (no scale/rotate on groups).
	•	Simplify: remove tiny segments and duplicate points if the source is noisy.
	•	Units/scale: OpenSCAD is unit-agnostic; use scale([sx,sy]) in SCAD if you need a physical size.

⸻

Output details
	•	points is a deduplicated, rounded list of 2D vertices.
	•	paths is a list of index arrays into points. Closed loops repeat the first index at the end.
	•	“Force close” ensures each open subpath is closed for reliable polygon winding.
	•	Y-axis can be inverted to match OpenSCAD’s coordinate system.

⸻

Bézier sampling
	•	The app samples cubic Béziers with uniform t and skips t=0 (so the start point isn’t duplicated).
	•	This keeps the vertex list lean and avoids zero-length edges; it does not change the geometry (the start point is already present).


⸻

Using the result in OpenSCAD

1) Use the points alone (debugging, measurements, maybe placing shapes at each point, etc.)

pts = [ [0,0], [10,0], [10,10], [0,10] ];

// Visualize vertices
for (p = pts) translate(p) circle(d=0.6);

// Convex hull of vertex markers (quick outline from points)
hull() for (p = pts) translate(p) circle(d=0.6);

2) Use points + paths in a polygon() (2D)

pts  = [ [0,0], [60,0], [60,40], [0,40], [15,10], [45,10], [45,30], [15,30] ];
cont = [0,1,2,3,0];       // outer rectangle
hole = [4,5,6,7,4];       // inner rectangle (hole)

polygon(points=pts, paths=[cont, hole], convexity=10);

3) Extrude to 3D

linear_extrude(height=3)
  polygon(points=pts, paths=[cont, hole], convexity=10);

4) Offsets, booleans, fillets (2D ops before extrude)

outline = offset(r=1) polygon(points=pts, paths=[cont], convexity=10);
difference() {
  linear_extrude(height=4) outline;
  translate([10,10,0]) cylinder(h=6, r=2, $fn=48);
}

5) Revolve a 2D profile

// If your profile is in the X–Y plane (right half), you can lathe it:
rotate_extrude(angle=360)
  polygon(points=profile_pts, paths=[profile_path], convexity=10);

6) Import vs. explicit polygon

If you don’t need the points, import("shape.svg") is quick—but you can’t access the vertices for custom logic. With this tool you can, and still linear_extrude() like normal. For SVG import tessellation controls see $fn/$fa/$fs.  ￼

⸻

Troubleshooting
	•	Nothing happens: make sure your SVG has <path> elements (convert shapes to paths).
	•	Wrong orientation: toggle Invert Y.
	•	Gaps/holes inverted: ensure subpaths are closed; try Force close and verify winding.
	•	Too many points: reduce Bézier segments or increase Decimals rounding.
	•	Transforms missing: apply them in your editor before exporting.

⸻
