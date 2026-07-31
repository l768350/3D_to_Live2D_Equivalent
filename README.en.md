English | [简体中文](README.md)

# 3D → Inochi2D OBJ Deformer

> This project was built with AI-assisted coding by an individual developer.

Automatically generates turn/roll deformer cages and parameter bindings for [Inochi2D](https://inochi2d.com/) puppet files (`.inx`) from a real 3D reference model (OBJ) — no more posing every angle by hand in Inochi Creator or eyeballing the depth.

In traditional Live2D/Inochi2D rigging, turn deformation usually relies on the artist manually estimating how far facial features should shift front-to-back to fake depth. This tool flips that around: give it a rough 3D model as "ground truth" for depth, and it samples the model's surface, computes the true projected coordinates after rotation, and writes them directly as Inochi2D deformer meshes plus Turn/Roll parameter bindings. Texture painting and fine facial adjustments are still left to you in Inochi Creator.

## Usage

Download `3D_to_Live2D_equivalent.zip` from this repo and unzip it.

### Desktop app (recommended)
```bash
pip install numpy pillow --break-system-packages
python3 app_tk.py
```
On Linux, if `tkinter` is missing: `sudo apt install python3-tk` (or the equivalent package for your distro).

If `python3` doesn't work, try `python app_tk.py` instead.

### Web app
```bash
pip install -r requirements.txt --break-system-packages
streamlit run app.py
```
If `python3` doesn't work, try `python -m streamlit run app.py` instead.

Both GUIs follow the same workflow: import an OBJ → (optional) import an existing `.inx` to add to it → (optional) correct the model's initial orientation first → for each part, fill in name/parent/parameters → confirm and write → save the `.inx`.

You can also use the core tool directly from the command line:
```bash
python3 add_obj_deformer.py --obj model.obj --out result.inx
# Append to an existing file, with a name and parent node
python3 add_obj_deformer.py --obj model.obj --in existing.inx --out result.inx --name "Head" --parent "Neck"
```
See `add_obj_deformer.py --help` for the full list of options.

## Features

- Three cage-building methods (`--cage-method`): `cylinder` (default — cylindrical unwrap around the Y axis, covers the full 360° of the model), `full-topology` (reuses the model's own mesh directly), `full-topology-uv` (reuses the model's mesh with its own UVs, so the model's own texture can be used directly)
- Split a model into independent parts by object name, material name, or both (`--group-by`) — parts are allowed to overlap
- Incrementally insert new nodes into an existing `.inx` without touching what's already there
- UV-connectivity analysis to diagnose whether a group is fragmented
- Two local GUIs (desktop via tkinter, web via Streamlit), plus a command-line interface

## Project Structure

| File | Purpose |
|---|---|
| `add_obj_deformer.py` | Core CLI: reads an OBJ and inserts deformer nodes + Turn/Roll bindings into a `.inx` |
| `obj_head_model.py` | OBJ parsing, ray-surface intersection, coordinate rotation, and other low-level geometry |
| `obj_pipeline.py` | Cage construction and parameter (keyframe) generation pipeline |
| `node_builder.py` | Helpers for building Inochi2D nodes/UUIDs |
| `inx_container.py` | Reading/writing `.inx` (JSON container) files |
| `gui_core.py` | Glue layer between the GUIs and the core logic; doesn't change core file behavior |
| `app_tk.py` | Desktop GUI (recommended) |
| `app.py` | Web GUI (Streamlit) |
| `anchors.py` / `mesh_gen.py` | Placeholder stubs left over from an earlier pipeline; unused by the current main flow |

## Known Limitations

- `full-topology-uv` mode doesn't generate a Roll parameter — add it manually in Inochi Creator
- `full-topology` (orthographic projection) mode doesn't support `--node-type part`
- Grouping by UV island (`--group-by island`) isn't wired into the main flow yet; the GUI's "diagnose" button shows the island breakdown, but you can't generate a node directly from one yet
