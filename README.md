# Auto-Tuned ICP

Auto-tuned Iterative Closest Point \(AICP\) is a module for non-incremental point cloud registration and localization failure prediction. The registration strategy is based on the libpointmatcher framework \(Pomerleau et al., AR 2012\).

AICP has been tested on Carnegie Robotics Multisense SL data from the NASA Valkyrie and Boston Dynamics Atlas humanoid robots, as well as the IIT HyQ quadruped and the Clearpath Husky mobile platform. The framework supports Lightweight Communications and Marshalling \(LCM\) integration for real-time message transfering.

### Details:

The algorithm:

1. accumulates planar laser scans on a thread and generates 3D point clouds with -b scans
2. on a second thread, stores the first cloud as the reference cloud
3. before alignment, overlap and alignability parameters --&gt; risk of alignment \(Nobili et al., ICRA 2018, submitted\) are computed
4. the reference cloud gets updated with latest accumulated cloud if \(risk of alignment &gt; threshold\)
5. aligns each new point cloud to the current reference cloud
6. publishes a corrected body pose

### Quick Start

The main dependencies are:

* [libpointmatcher](https://github.com/ethz-asl/libpointmatcher.git), a modular library implementing the Iterative Closest Point \(ICP\) algorithm for aligning point clouds. The most recent revisions of this codebase \(from July 2017 onwards, with the introduction of the Failure Prediction module\), depend on our [own fork of libpointmatcher](https://github.com/oxfordrobotics/libpointmatcher/commit/3393c9327677c649d480799e76159ea223d95004) \(revision 3393c9327677c649d480799e76159ea223d95004 on branch sn-system-covariance\).

* [Point Cloud Library \(PCL\)](https://github.com/pointcloudlibrary/pcl) revision pcl-1.8.0, a standalone, large scale, open project for 2D/3D image and point cloud processing.

* [Octomap](https://github.com/OctoMap/octomap.git), a probabilistic 3D mapping framework based on octrees.

#### Running

Help: `aicp-registration-online -h`

```
Usage:
  aicp-registration-online [opts]
Options:
  -h,  --help                     = [false]                 : Display this help message
  -s,  --working_mode             = ["robot"]               : Robot or Debug? (i.e. robot or debug)
  -u,  --failure_prediction_mode  = [0]                     : Use: Alignment Risk (0), Degeneracy (1), ICN (2)
  -v,  --verbose                  = [false]                 : Publish frames and clouds to LCM for debug
  -c,  --apply_correction         = [false]                 : Initialize AICP with corrected pose? (during debug)
  -pc, --pose_body_channel        = ["POSE_BODY"]           : Input kinematics-inertia pose estimate
  -oc, --output_channel           = ["POSE_BODY_CORRECTED"] : Output corrected pose message
  -l,  --lidar_channel            = ["MULTISENSE_SCAN"]     : Input planar laser scan message
  -b,  --batch_size               = [240]                   : Number of scans per full 3D point cloud (at 5RPM)
  -m,  --min_range                = [0.500000]              : Closest accepted lidar range
  -M,  --max_range                = [15.0000]               : Furthest accepted lidar range
```

Example Usage: `aicp-registration-online -s debug -b 83 -a -v`

**Note:**

* Option _"-s robot"_ is meant to be used if the corrected pose from AICP is fed back into the state estimator. Each corrected pose is published only once as soon as computed.
* Option _"-s debug"_ is meant to be used during debug. The corrected pose message is published at the same frequency as the pose estimate from the state estimator \(for visualization purposes\).

### Credits

This work has been accepted for publication in the Proceedings of 2017 IEEE International Conference on Robotics and Automation \(ICRA\):

```
@inproceedings{Nobili17icra,
  title  = {Overlap-based {ICP} Tuning for Robust Localization of a Humanoid Robot},
  author = {S. Nobili and R. Scona and M. Caravagna and M. Fallon},
  booktitle = {{IEEE International Conference on Robotics and Automation (ICRA)}},
  location = {Singapore},
  month = may,
  year   = {2017},
}
```

The following paper has been submitted for review to 2018 IEEE International Conference on Robotics and Automation \(ICRA\):

```
@inproceedings{Nobili18icra,
  title  = {Predicting Alignment Risk to Prevent Localization Failure},
  author = {S. Nobili and G. Tinchev and M. Fallon},
  booktitle = {{IEEE International Conference on Robotics and Automation (ICRA)}},
  month = may,
  year   = {2018},
  note =    {Submitted},
}
```

**Since this work is still under review, github users who have been granted access to this repository are not allowed to share the content. Simona.**

### License

The License information is available in the LICENSE file contained in this project repository.

Simona Nobili, Nov 2017.  
Email: simona.nobili@ed.ac.uk, snobili@robots.ox.uk
