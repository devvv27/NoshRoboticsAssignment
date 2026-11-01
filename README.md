# Egg segmentation assignment

This repository contains work for the four-task computer vision assignment. The image dataset is used for Tasks 1–3. The supplied egg-spinning video is used for Task 4.

## What is here

| Folder or file | What it contains |
|---|---|
| `Task1/` | Notebooks for segmenting eggs with a pretrained model. The current preferred notebook is `task1_secondTry.ipynb`. It uses Grounding DINO to find egg boxes and SAM to make masks. |
| `Task1/task1_results/` | Saved Task 1 masks and visual previews, including the `combined_masks` used as reference labels later. |
| `Task2/` | Three versions of the classical image-processing notebook. The current preferred version is `task2_ThirdTry.ipynb`. |
| `Task3/task3_evaluation.ipynb` | Compares Task 2 predictions with Task 1 masks and displays scores, error plots, and difficult examples. |
| `Task4/task4_video_tracking.ipynb` | Runs the Task 2 method on the video and links egg detections over time. |
| `Dataset/` | Roboflow egg images used in Tasks 1–3. |
| `Assignment.pdf` | The task instructions. |
| Root MP4 video | Video supplied for Task 4. |

## Progress so far

| Task | Work done | Main result or next step |
|---|---|---|
| 1 — pretrained segmentation | Built a Grounding DINO + SAM workflow and saved masks and previews. | Use the second-try notebook and its `combined_masks` as the Task 1 reference set. |
| 2 — classical segmentation | Tried thresholds, colour rules, candidate blobs, GrabCut, and an ellipse search. The final notebook records the trials and saves predictions, metrics, settings, and a zip. | The third-try notebook is the current final version. The saved run is available under Task2/task2_results/task2_results/, including masks, metrics, and the final config. |
| 3 — evaluation | Notebook is prepared to compare saved Task 2 predictions against Task 1 masks. | Task 3 is set up for local use and finds the repository dataset and Task 1 masks automatically. It also detects the saved Task 2 outputs under Task2/task2_results/task2_results/. |
| 4 — video tracking | Notebook is prepared to segment video frames and associate detections between frames. | Run Task 2 first so Task 4 can load its saved settings, then run the video notebook. |

## Task 2 experiments in plain English

The scores below are mean IoU on the 87-image development split, as recorded in the final Task 2 notebook. Higher means more overlap with the Task 1 masks. These are comparisons with model-generated masks, not hand-labelled truth.

| Experiment | What was tried | What happened |
|---|---|---|
| T01–T04 | Basic brightness and colour thresholds, then GrabCut started from a box near the image centre. | The simple thresholds picked up other bright objects. Centre-started GrabCut was the strongest baseline at about 0.556 IoU. |
| T05–T09 | Find blobs from several threshold methods and rank them by egg-like shape and colour. Adjust extra-egg and whiteness rules. | The scorer often preferred a bowl, tray, or cloth blob. Tuning helped some, but this approach did not beat the best baseline. |
| T10–T13 | Refine masks with GrabCut, fit ellipses, and change morphology. | These changes made little difference to the best baseline. False-positive regions were still the main issue. |
| E01–E05 | Search for ellipse outlines, then adjust allowed sizes, extra detections, and interior texture penalty. | The approach started lower, but selected eggs more precisely. Larger minimum sizes and texture checks helped reject clutter. |
| E06–E12 | Refine each ellipse with GrabCut, and try different refinement settings. | GrabCut in a limited band around the ellipse improved the result. Forcing only one egg made results worse, so the method keeps multi-egg detections. |
| E13–E19 | Investigate errors where small ellipse-shaped distractors were mistaken for eggs. Try contrast and size settings. | The size prior gave a large improvement. The final stronger size reward scored about 0.756 IoU, only just above the previous setting, so that tiny gain may be noise. |

The final method looks for likely egg-shaped outlines, uses colour contrast and texture to rank them, applies a size preference, and refines the boundary with GrabCut. See the experiment log in `task2_ThirdTry.ipynb` for each trial and its keep/discard decision.

## Running Tasks 2–4

1. Open `Task2/task2_ThirdTry.ipynb` and provide the `Dataset/` images and Task 1 `combined_masks/` as inputs. In Kaggle, the notebook writes to `/kaggle/working/task2_results` and creates `task2_results.zip`.
2. Save or attach the `task2_results` folder. Task 3 needs its `pred_masks/`; Task 4 needs its `final_config.json`.
3. Open `Task3/task3_evaluation.ipynb`. Make the Task 1 and Task 2 result folders visible, or set `TASK1_MASK_DIR` and `TASK2_DIR` in the first cell.
4. Open `Task4/task4_video_tracking.ipynb`. Make the Task 2 result folder visible. If the video is not beside the notebook, set `EGG_VIDEO` to its path. The notebook saves an annotated MP4 and `tracks.csv` under `task4_output/`.

The notebooks use Python with OpenCV, NumPy, pandas, Matplotlib, tqdm, and SciPy (SciPy is used for Task 4 tracking). Task 1 additionally needs its Grounding DINO and SAM setup; see that notebook for details.

## Important notes

- Task 1 masks are treated as reference masks in Tasks 2 and 3. They can have errors because they were generated by a model.
- The Task 2 evaluation split is small. Its notebook notes that evaluation results had been viewed before the final development-only tuning, so treat them as a slightly optimistic estimate.
- Video tracking uses nearest-centre matching. Occlusion, scene cuts, or large jumps can cause a new ID or an ID switch.
- The assignment asks for a short disclosure of AI coding tool use in the final report.
