# Stop Sign Annotation Protocol

## Summary

- We are working on training a machine learning model to analyze driver behavior around stop signs using dash cam video footage.
- Attached are a few short video clips, automatically flagged by a detection algorithm as potentially involving stop signs. Your task is to design an annotation protocol for remote annotators, with the aim of extracting the most useful training signal possible from this data.
- You may assume that the only available information is the video itself. There is no access to metadata such as speed, GPS, or external sensors.

### Deliverables

1. A proposed set of labeling instructions, written clearly and concisely, as you would send to remote annotators.
2. Any assumptions or tradeoffs you make in the design.
3. A short explanation of how your labeling scheme supports training an effective model for this task.

---

## 1. Instructions for Remote Annotators

### 1.1 Overview

You will receive dash cam video clips in which at least one stop sign may appear. Your overall goals are:

1. **Identify the intervals of time** (in the video) where any stop sign is visible.
2. **Label the driver’s stopping behavior** (e.g., full stop, rolling stop) over the relevant time interval in the clip.
3. **Produce segmentation maks** for each stop sign that appears during those intervals (on a subset of frames, to reduce workload).

In many cases, you will see:

- Multiple stop signs visible at different times or in different parts of the frame.
- Varying degrees of occlusion, lighting, or distance.
- Possible partial or truncated views if the video starts or ends mid-interaction.

### 1.2 Step-by-Step Annotation Process

#### 1. Watch the Entire Clip to Identify Key Time Intervals

1. **Scan the video** from start to finish and note any periods (intervals) in which you see a stop sign, even if it’s partially visible or far away.
2. **Mark each interval** using the annotation tool’s timeline editor as "**Stop Sign Interval A**", "**Stop Sign Interval B**", etc., if multiple signs appear.
3. If the clip contains no recognizable stop signs, mark the entire clip as "**No Stop Sign Visible**".

#### 2. Mark the Driver’s Reaction Interval

1. For each "**Stop Sign Interval**", identify the timeframe in which the vehicle appears to be responding to the relevant stop sign (e.g., braking, coming to a stop, or passing through).
2. If the driver does **not react at all** (e.g., does not slow or stop), still mark a "**Driver Response Interval**" that covers the time when the vehicle passes the stop sign, and label it as "**No Stop**".
3. If the video ends before you can determine the driver’s reaction, mark the interval as "**Not Determinable**".

#### 3. Label the Driver Behavior (Interval-Level)

Using the "**Driver Response Interval**" choose one of these labels for classifying each interval:

- **Full Stop** – The vehicle clearly halts forward motion for ~1 second (i.e., ~30 frames if 30 FPS).
- **Rolling Stop / Incomplete Stop** – The vehicle slows but does not fully stop.
- **No Stop** – The vehicle neither slows nor halts for the sign.
- **Not Determinable** – The video is too short, unclear, or ends before the action is complete.

#### 4. Segmentation Mask Annotation (Frame-Level)

From the different "**Stop Sign Intervals**" you selected our system will sample select **key frames** at a regular sampling rate (e.g., every 3rd frame):

1. On these sampled frames, **segment** the shape around the visible stop sign. Keep it as tight as possible to the edges of the sign. (Our system should provide oval shape selector, to facilitate this task).
2. If the sign is partially visible or mostly blocked, **segment it** if you can confirm it’s a stop sign.

#### 5. Visibility Labeling (Frame-Level)

For each **segmentation mask** you have annotated for the different stop signs you need to select one of the following classes:

- **Clear** – The sign face is fully visible (~90%+ unobstructed).
- **Partially Covered** – The sign is visible but somewhat blocked by an object, reflection, etc.
- **Heavily Covered** – The sign is mostly obstructed yet still identifiable.

---

## 2. Assumptions & Tradeoffs

### 2.1. Interval-Based Annotation

Breaking the clip into intervals of sign visibility dramatically reduces time spent labeling. Annotators do not need to segment frames where no sign appears, focusing effort only on relevant video portions. This also helps handle edge cases, such as videos that start or end mid-interaction, or those with multiple stop signs.

### 2.2. Subsampling Key Frames

Instead of segmenting every frame in a 40-second clip (which could be 1,200 frames at 30 FPS), our system selects key frames from the annotated intervals at a consistent sampling rate. This balances annotation workload and model training needs, while still capturing enough variation in sign appearance and visibility.

### 2.3. Segmentation

We require annotators to **segment the actual shape of the stop sign** (using a mask, ideally with an oval selector), rather than drawing a simple bounding box. This provides more precise data for training, especially in cases of partial occlusion or unusual sign orientation. However, segmentation is more time-consuming than bounding boxes, so we mitigate this by subsampling frames and only sampling for the selected intervals.

### 2.4. Multiple Timelines

- One (or more) timeline track(s) for the stop sign intervals.
- One (or more) timeline track(s) for the driver’s reaction.
- A final "clip-level" label if the clip does not present any important information.

### 2.5. One-Second Stop

"**Full Stop**" is based on visually confirming ~1 second of complete motion halt.

### 2.6. Handling Uncertainty and Edge Cases

- If the clip is too short or ends before the action completes, mark as "**Not Determinable**". This helps maintain annotation quality when details are missing.
- If **no stop sign is visible** at any point, annotators should mark the entire clip as "**No Stop Sign Visible**" and skip all further annotation steps (no segmentation, no driver behavior labeling).
- If a stop sign is visible but the driver does **not react** (does not slow or stop), annotators should still mark a "**Driver Response Interval**" covering the time when the vehicle passes the stop sign and label it as "**No Stop**". This ensures the model learns from both compliant and non-compliant driver behaviors.

---

## 3. Labeling Scheme Justification

### 3.1. Efficient Data Collection

Interval-based labeling and frame subsampling significantly reduce annotators’ effort, making it feasible to label longer clips without sacrificing data quality.

### 3.2. Explicit Driver Behavior Annotation

A separate interval for driver behavior (e.g., braking vs. continuing) ties the presence and visibility of a sign to the vehicle’s actual reaction.

### 3.3. Scalability & Consistency

Remote annotators can quickly mark intervals and only label the frames inside those intervals. This approach scales well across many videos, keeping instructions straightforward yet comprehensive.

### 3.4. Robust to Multiple Signs & Partial Clips

By creating multiple sign intervals and separate driver reaction intervals, we maintain clarity when multiple signs appear in the same clip or when the video is truncated.

### ### 3.5. Rich Training Signal

By combining interval, frame, and behavior-level labels, the protocol enables training models that can:

- Detect stop signs under varied conditions (distance, occlusion, lighting).
- Infer driver compliance with traffic rules.
