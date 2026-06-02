# Precalculus Limits with Manim 🚀

Welcome! This workspace contains a Jupyter Notebook designed for teaching limits to an IB middle school/early high school student using the magical **Manim** animation engine (the same one used by 3Blue1Brown!).

## 📂 Included Materials

**[Intro_to_Limits.ipynb](file:///c:/Users/ADIB/OneDrive/Desktop/FarmVerse/Intro_to_Limits.ipynb)**: The interactive lesson notebook containing:
   - `%%manim` cells to generate high-quality math animations directly in the notebook.
   - An animation for **Zeno's Walk** (intuition of a limit).
   - An animation for a **Removable Discontinuity** (the "Hole" at $x=1$).
   - An animation for a **Jump Discontinuity** (Left vs. Right limits).
   - Symbolic math verification using `SymPy`.

## 🛠️ Quick Setup Guide

We have installed the necessary Python packages and the FFmpeg system package on your computer.

### Running the Notebook:
1. Open the [Intro_to_Limits.ipynb](file:///c:/Users/ADIB/OneDrive/Desktop/FarmVerse/Intro_to_Limits.ipynb) file in VS Code or Jupyter.
2. Click **"Restart"** on your Jupyter kernel at the top right of the notebook.
3. Run the first cell. It has auto-detection code to dynamically find the FFmpeg binary location on your system and add it to the environment.
4. Run each subsequent code cell! The animations will render and embed video players directly under the cells.

### Manual Setup (For other machines):
If you set this up on another Windows machine:
1. Install Python packages: `pip install manim imageio-ffmpeg sympy`
2. Install FFmpeg via winget: `winget install Gyan.FFmpeg`
