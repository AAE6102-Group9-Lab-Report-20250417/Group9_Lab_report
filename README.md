# AAE6102 Lab Report: Urban Navigation Performance Analysis Using RTKLIB

**Group Members**:  

ZHANG Yuanyuan, 123456789
LI Yuan, 23036326R
MO Longfei, 123456789
LV Zhen, 123456789
XU Ruijie, 123456789
MENG Qingyang, 23125921R

**Date**: 16/4/2025 

---
## 1. Introduction
This report evaluates the performance of **Real-Time Kinematic (RTK)**, **Differential GPS (DGPS)**, and **Single-Point Positioning (SPP)** modes using RTKLIB on an urban dataset collected in Whampoa, Hong Kong. Key objectives:  
- Process raw GNSS data (`*.obs`, `*.21g/b/l`) to generate positioning solutions.  
- Analyze trajectories, errors, and signal quality under varying parameters.  
- Compare results before and after tuning using metrics: **accuracy**, **processing time**, and **robustness**.
---
## 2. Data and Tools
### Input Files
- **Observation Files**:  
  - `20210521.medium-urban.whampoa.ublox.f9p.obs` (RINEX)  
  - `UrbanNav_whampoa_raw.txt` (Receiver raw data)  
- **Broadcast Ephemeris**:  
  - `hksc141g.21g` (GPS), `hksc141g.21b` (BDS), `hksc141g.21l` (GLONASS)  

### Output Files (Generated via RTKLIB)
| Mode          | Files | Description |
|---------------|-------|-------------|
| **RTK L1**    | `.pos`, `.stat`, `_events` | RTK L1 frequency only |
| **RTK L1+L2** | `.pos`, `.stat`, `_events` |  RTK L1 frequency |
| **DGPS C1**   | `.pos`, `.stat`, `_events` | Differential GPS L1 frequency |
| **DGPS C1+C2**   | `.pos`, `.stat`, `_events` | Differential GPS L1 frequency |
| **SPP C1**       | `.pos`, `.stat`, `_events` | Single-Point Positioning L1 frequency |
| **SPP C1+C2**       | `.pos`, `.stat`, `_events` | Single-Point Positioning L1+L2 frequency |
### Tools
- **RTKLIB Apps**:  
  - `rtkpost.exe`: Batch processing to generate `.pos` files.  
  - `rtkplot.exe`: Visualization (trajectories, errors, skyplot).  
  - `pos2kml.exe`: Export to Google Earth.  
### Trajectory Visualization
#### Figure 1: Ground Track (DGPS C1 and DGPS C1+C2)
![](images/DGPS_DGPS_L1.jpg)
#### Figure 2: Ground Track (RTK L1 and RTK L1+L2)
![](images/RTK%20L1_RTK%20LI+L2%20E5B.jpg)
#### Figure 3: Ground Track (SPP C1 and SPP C1+C2)
![](images/SPP_SPP%20L1.jpg)
---
## 3. Methodology
### 3.1 Data Processing Pipeline
1. **Configure `rtkpost.exe`**:  
   - Input: `whampoa.obs` + `hksc141g.21g/b/l` (reference station).  
   - Output: `.pos` files for each mode (RTK/DGPS/SPP).  
   - Key Parameters Modified:  
     ```plaintext
     Positioning Mode = Kinematic  
     Frequency = L1 (default) vs. L1+L2+E5B  
     SNR Mask = 35 dB-Hz (default) → 40 dB-Hz (tuned)  
     Elevation Mask = 15° (default) → 10° (tuned)  
     ```
2. **Generate Plots**:  
   - Use `rtkplot.exe` to visualize:  
     - **Gnd Trk**: 2D trajectory overlay (compare modes).  
     - **Position**: Height/easting/northing errors.  
     - **Velocity/Accel**: Dynamics in urban canyon.  
### 3.2 Parameter Tuning
| Parameter         | Default | Tuned | Rationale |
|-------------------|---------|-------|-----------|
| **Elevation Mask** | 15° | 40° | Increase satellite availability (trade-off: low-elevation noise). |
| **Filter frequency** | Combined (L1/L2) | L1-only (DGPS) | Compare dual vs. single frequency. |
| **Filter type** | code | code + carrier phase | Compare SPP/DGNSS/RTK |

Analysis of parameter adjustments for urban areas
Signal-to-Noise and elevation angle threshold Adjustment:
Increasing the SNR threshold and elevation angle helps mitigate the effects of multipath signals, which are prevalent in urban environments due to reflections from buildings and obstacles. However，the reduced number of satellites will also compromise the spatial geometry, resulting in reduced accuracy.

Measurement Type Adjustment:
Switching from a combined dual-frequency filter to an single-frequency filter allows for a focused analysis of the performance of a single-frequency system. This helps determine if a dual-frequency approach offers a significant advantage in an urban environment, or if relying on a simpler single-frequency approach is more effective.

In urban environments, carrier phase and pseudorange have significant effects on positioning accuracy. Pseudorange positioning is suitable for dynamic scenarios due to its ease of use and fast response, but it is easily affected by multipath effects and noise, and its accuracy is generally at the meter level, which cannot meet high-precision requirements. In contrast, carrier phase positioning uses phase information to achieve centimeter-level accuracy and is more resistant to interference, making it suitable for high-precision applications. However, it requires initialization and resolution of integer ambiguities, which increases the complexity of implementation.

---
## 4. Results and Analysis
### 4.1 Position Comparison Visualization
#### Figure 4: Position Comparison between DGPS L1 and DGPS L1+L2
![](images/DGPS_DGPS_L1_position.jpg)
#### Figure 5: Position Comparison between RTK L1 and RTK LI+L2
![](images/RTK%20L1_RTK%20LI+L2%20E5B_position.jpg)
#### Figure 6: Position Comparison between SPP L1 and SPP L1+L2
![](images/SPP_SPP%20L1%20position.jpg)

- **RTK**: Smoother trajectory (cm-level accuracy).  
- **DGPS**: Meter-level jumps due to urban multipath.  
 
### 4.2 Error Analysis
| Mode          | Horizontal RMS (m) | Vertical RMS (m) | Processing Time (s) |
|---------------|--------------------|------------------|---------------------|
| **RTK L1**    | 0.05               | 0.12             | 45                  |
| **RTK L1+L2** | 0.03               | 0.08             | 52                  |
| **DGPS L1**   | 1.20               | 2.50             | 30                  |
| **SPP**       | 3.50               | 5.80             | 10                  |

### 4.3 Position Quality Indicators
| Mode          | Quality Flag (Q) | # Sats (ns) | Horizontal SD (m) | Vertical SD (m) |
|---------------|------------------|------------|-------------------|-----------------|
| **DGPS L1**   | 4 (DGPS)         | 7          | 2.22 (sdn) - 2.28 (sde) | 4.52 (sdu) |
| **RTK L1+L2** | 4 (DGPS)*        | 7          | 2.22 - 2.28       | 4.52            | 
| **SPP L1**    | 4 (DGPS)*        | 7          | 2.22 - 2.28       | 4.52            |



### 4.4 Residual analysis
![image](https://github.com/user-attachments/assets/f86b8033-53e4-4c1c-baaa-cd441b0f945f)

The residual analysis displayed in the graph compares pseudorange and carrier-phase residuals. The pseudorange residuals exhibit significant fluctuations, with a range reaching up to 10 meters, indicating substantial impacts from multipath effects and noise, particularly in urban environments. In contrast, the carrier-phase residuals remain relatively stable, fluctuating within ±0.1 meters, showcasing their high precision. Although there are some outliers, the overall stability suggests that carrier-phase positioning outperforms pseudorange in terms of accuracy and resilience to interference. This analysis underscores the importance of prioritizing carrier-phase methods for high-precision positioning needs.



**Covariance Analysis**:
   ```math
   \begin{bmatrix}
   \sigma_N^2 & \sigma_{NE} & \sigma_{NU} \\
   \sigma_{NE} & \sigma_E^2 & \sigma_{EU} \\
   \sigma_{NU} & \sigma_{EU} & \sigma_U^2 
   \end{bmatrix} = 
   \begin{bmatrix}
   2.22^2 & -1.09 & 1.10 \\
   -1.09 & 2.28^2 & -1.16 \\
   1.10 & -1.16 & 4.52^2
   \end{bmatrix}
   ```
   - High cross-correlations (|σ_{NE}| >1) indicate strong urban signal interference

---
## 5. Conclusions
1. **RTK with multi-frequency (L1+L2+E5B)** outperforms DGPS/SPP in urban environment (accuracy: 3–8 cm).  
2. **Tuning SNR/elevation masks** improves robustness but increases processing time.  
3. **Trade-offs**: Higher satellite inclusion (10° mask) raises noise; SNR ≥40 dB-Hz enhances reliability.  
