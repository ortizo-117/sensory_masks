# Mask Overlap Heatmaps

This repository contains a Jupyter notebook for creating overlap heatmaps from black-and-white subject mask PNGs. The notebook loads masks from annotation folders, converts the selected mask color into boolean arrays, computes pixel-by-pixel overlap, and exports colored PNG heatmaps.

The main workflow is in [`mask_overlap_heatmaps.ipynb`](mask_overlap_heatmaps.ipynb).

![Example overlap heatmap](export%20lumbar%20vanc/overlap_outputs/ano_2_overlap_percent_with_outline.png)

## What The Notebook Does

The notebook is designed for collections of subject masks where each PNG has the same dimensions and uses black or white pixels to mark an anatomical region.

It can:

- Load all PNG masks from one annotation folder.
- Count how many subject masks overlap at each pixel.
- Convert overlap counts into percentages of subjects.
- Generate transparent, white, or black background heatmap PNGs.
- Apply region-specific color palettes for `L5`, `L5S1`, and `S1`.
- Overlay an anatomy outline image on top of each heatmap.
- Optionally draw individual subject boundaries on top of the group overlap heatmap.
- Save output images into `export lumbar vanc/overlap_outputs/`.

## Project Structure

```text
Peter_masks/
|-- mask_overlap_heatmaps.ipynb
|-- requirements.txt
|-- anatomy_only_sensory (1).png
|-- anatomy_only_pain (1).png
`-- export lumbar vanc/
    |-- ano_1/
    |   `-- sensory__segment_1__case_*.png
    |-- ano_2/
    |   `-- sensory__segment_2__case_*.png
    |-- ano_3/
    |   `-- sensory__segment_3__case_*.png
    `-- overlap_outputs/
        `-- generated heatmap PNGs
```

By default, the notebook treats the folders as:

| Folder | Region | Palette |
| --- | --- | --- |
| `ano_1` | `L5` | Blue |
| `ano_2` | `L5S1` | Purple |
| `ano_3` | `S1` | Pink/red |

These assignments are controlled by `ANO_REGION_MAP` and `REGION_PALETTES` in the notebook.

## Setup

1. Clone or download this repository.

2. Create and activate a Python environment.

   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

   On macOS or Linux:

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

3. Install the required packages.

   ```bash
   pip install -r requirements.txt
   ```

4. Start Jupyter.

   ```bash
   jupyter notebook
   ```

5. Open `mask_overlap_heatmaps.ipynb`.

## How To Use The Notebook

Run the notebook cells from top to bottom.

The setup cell defines the input and output locations:

```python
ROOT = Path("export lumbar vanc")
ANATOMY_OUTLINE = Path("anatomy_only_sensory (1).png")
OUTPUT_DIR = ROOT / "overlap_outputs"
```

The input folders should contain one PNG per subject. Every PNG in a folder must have the same width and height.

### Process One Folder

The "Process One Folder" section creates a count-based heatmap for one annotation folder:

```python
result_ano_1 = create_overlap_heatmap(
    ROOT / "ano_1",
    output_path=OUTPUT_DIR / "ano_1_overlap_count_with_outline.png",
    mode="count",
    cmap=palette_for_folder(ROOT / "ano_1"),
    background="transparent",
    outline_path=ANATOMY_OUTLINE,
)
```

Use this section when you want to test one folder before processing all annotations.

### Process All Folders

The "Process All `ano_*` Folders" section loops through every annotation folder and creates one count heatmap per folder:

```python
for folder in sorted(ROOT.glob("ano_*")):
    create_overlap_heatmap(folder, mode="count")
```

This is useful when all annotation folders are already organized inside `export lumbar vanc/`.

### Percent Overlap

The "Percent Overlap Instead Of Count" section creates heatmaps where colors represent the percentage of subject masks overlapping at each pixel:

```python
mode="percent"
percent_step=20
percent_color_range="auto"
```

Use percent mode when folders have different numbers of subject masks and you want outputs that are easier to compare across regions.

### Subject-Specific Outlines

The "Subject-Specific Outlines" section keeps the group overlap heatmap but adds boundary lines from each individual subject mask:

```python
subject_outlines=True
subject_outline_alpha=0.50
subject_outline_width=3
```

This helps show whether a heatmap hotspot comes from tightly aligned masks or from broader subject-to-subject variation.

## Output Files

Generated images are saved in:

```text
export lumbar vanc/overlap_outputs/
```

Common output names include:

- `ano_1_overlap_count_with_outline.png`
- `ano_2_overlap_count_with_outline.png`
- `ano_3_overlap_count_with_outline.png`
- `ano_1_overlap_percent_with_outline.png`
- `ano_2_overlap_percent_with_outline.png`
- `ano_3_overlap_percent_with_outline.png`
- `ano_1_overlap_percent_with_subject_outlines.png`
- `ano_2_overlap_percent_with_subject_outlines.png`
- `ano_3_overlap_percent_with_subject_outlines.png`

## Customization

You can adjust the notebook for different mask sets:

- Change `ROOT` if your input folders are somewhere else.
- Change `ANATOMY_OUTLINE` to use another outline image, such as `anatomy_only_pain (1).png`.
- Change `mode` to `"count"` for raw subject counts or `"percent"` for normalized overlap.
- Change `background` to `"transparent"`, `"white"`, or `"black"`.
- Change `threshold` if your PNG masks are anti-aliased or not strictly black and white.
- Change `mask_color` to `"white"` if the selected region is white instead of black.
- Change `percent_step` to control percent bin sizes, or set it to `None` for a continuous gradient.
- Edit `REGION_PALETTES` to use different colors.
- Edit `ANO_REGION_MAP` if folder names should map to different anatomical regions.

## Troubleshooting

If the notebook raises `No PNG files found`, check that the folder path is correct and that the files end in `.png`.

If the notebook raises an image dimension error, one or more masks in the same annotation folder has a different size. Resize or re-export the masks so all files in that folder match.

If the heatmap appears empty, check whether the masks use black or white for the selected region. The current notebook default is:

```python
mask_color="black"
```

If the anatomy outline fails to overlay, make sure the outline PNG has the same dimensions as the mask PNGs.

## Dependencies

The notebook uses:

- `numpy`
- `pillow`
- `matplotlib`
- `jupyter`

Install them with:

```bash
pip install -r requirements.txt
```
