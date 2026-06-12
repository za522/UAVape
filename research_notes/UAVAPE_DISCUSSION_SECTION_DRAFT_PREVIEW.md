# UAVape Discussion Section Draft Preview

## Discussion

The final UAVape detector should be interpreted as the outcome of a controlled validation-based selection process rather than as an isolated YOLO training result.

The held-out test result indicates that the selected detector generalised well to unseen real images. The final YOLO26s@1280 checkpoint achieved a vape AP50 of 0.882 and a vape AP50--95 of 0.685 on the sealed test split, with aggregate two-class AP50 of 0.851 and AP50--95 of 0.638. The reduction in vape AP50 from validation to test suggests that the test split contained harder or more varied vape instances, but the retained AP50--95 score shows that localisation quality did not collapse under the real-only held-out evaluation.

The model-ablation results suggest that UAVape performance was driven less by model capacity alone and more by preserving small-object detail. The selected YOLO26s@1280 configuration outperformed larger 640-pixel alternatives on the main vape metrics, indicating that input resolution was a critical factor for disposable vape detection. This supports the interpretation that vapes are not merely a class-recognition problem, but a small-object localisation problem where downsampling can remove useful visual evidence.

The data-source ablation also shows that the benefit of external data was not simply a function of adding more images. Scraped-only and synthetic-only additions produced mixed, non-monotonic behaviour, while the combined scraped-plus-synthetic recipe produced the strongest validation result. This suggests that complementary source diversity was more useful than volume from a single external source. In practical terms, the DCE's value was not only that it increased dataset size, but that it allowed source-controlled experiments to identify which combinations of data were actually beneficial.

The dataset contribution should also be interpreted separately from the model score. UAVape is smaller than broad general-purpose litter corpora, but it is more tightly targeted: the final real dataset contains 1,204 real images and 2,040 labelled object instances, including 1,328 vape boxes and 712 lighter boxes. Its contribution is therefore not only scale, but specificity: it isolates disposable vapes as a hazardous micro-WEEE class while retaining a visually similar confuser class for evaluation.

| Dataset component | Scale / composition | Discussion relevance |
|---|---|---|
| UAVape real dataset | 1,204 images; 2,040 boxes; 1,328 vape boxes; 712 lighter boxes | Dedicated vape-specific micro-WEEE dataset with an auxiliary lighter confuser class. |
| UAVape sealed test split | 184 real images; 420 boxes; 255 vape boxes; 165 lighter boxes | Held back from model/data selection and used once for final generalisation reporting. |
| Final selected training recipe | Real training data plus 355 scraped and 355 synthetic vape images | Demonstrates how the DCE enabled source-controlled augmentation rather than indiscriminate data addition. |
| Reviewed litter datasets | Broader rubbish, bottle, plastic, trash or waste classes; varying annotation formats and split protocols | Useful as contextual benchmarks, but not directly scale- or target-matched to vape-specific detection. |

| Study / dataset | UAV? | Reported metric | Task / target | Comparison nuance |
|---|---|---|---|---|
| UAVape V4 | Yes / UAV-oriented | Test vape AP50 = 0.882; all AP50 = 0.851 | Disposable vapes with lighter confuser class | Held-out real test split; modern YOLO26s@1280; source-controlled data ablation. |
| UAVVaste | Yes | YOLOv4 mAP@50 = 0.785 | One-class rubbish detection | UAV setting and small objects are relevant, but target is generic rubbish rather than vape-specific micro-WEEE. |
| SODA | Yes | YOLOv8 mAP approx. 0.796 | Multi-class aerial litter | Strong UAV comparison; uses tiling and multiple litter classes, so not a direct target-class comparison. |
| HAIDA | Yes | YOLOv4-Tiny AP50 above 0.70 | UAV marine trash / bottle-type objects | Real-time UAV orientation, but older lightweight detector and limited reported evaluation detail. |
| BDW | Yes | Faster R-CNN AP approx. 0.903 | Plastic bottle detection | High AP, but bottle-only, oriented boxes and tiled UAV imagery make direct comparison difficult. |
| TACO | No | YOLOv5x AP50 approx. 0.633 | Ground-level multi-class litter | Broader and more heterogeneous litter dataset; non-UAV, but useful for natural-scene litter comparison. |
| PlastOPol | No | YOLOv5x AP50 approx. 0.849 | Outdoor plastic/litter detection | Similar numerical AP50 to UAVape aggregate AP50, but larger-object and non-UAV differences matter. |
| TrashNet | No | YOLOv5 mAP@50 approx. 0.950 | Controlled indoor waste images | Higher score but controlled non-UAV setting with much simpler visual conditions. |

This comparison should not be read as a direct leaderboard because the reviewed studies differ in target class, dataset domain, annotation format, split protocol, input resolution and whether the reported value is validation, test or cross-dataset performance. However, the comparison shows that UAVape V4 is numerically competitive with several UAV litter-detection benchmarks while addressing a narrower and more hazardous target class. The strongest claim is therefore not that UAVape universally outperforms prior work, but that vape-specific micro-WEEE detection can achieve competitive held-out performance when dataset construction, model selection and data-source composition are treated as controlled experimental variables.

The result is particularly notable because disposable vapes are visually small, low-salience and easily confused with lighters or other cylindrical litter. In this context, the stronger vape-class AP50 relative to the aggregate two-class AP50 suggests that the model learned the primary target class more effectively than the auxiliary confuser class. The lighter class should therefore be interpreted less as a second deployment objective and more as a diagnostic mechanism for hard-negative boundary learning.

The lighter class also had a methodological purpose during training. If visible lighters appeared in images but were left unlabelled, they would become contradictory background supervision: the model would see a salient vape-like object while the label file implied that no object was present. Labelling lighters as an auxiliary class made this ambiguity explicit and allowed the benchmark to test whether the detector had learned vape-specific cues rather than a generic elongated-litter shape.

The negative SAHI result is still informative. Tiling was tested as a small-object inference strategy because slicing can increase the apparent scale of small objects within each crop. However, the tested configuration did not improve the primary vape metrics and introduced additional duplicate- or false-positive pressure. This suggests that, for UAVape, full-frame high-resolution inference preserved useful scene context better than the tested static tiling setup.

Several limitations remain. The test set is still modest in size, the comparison to literature is constrained by inconsistent reporting protocols, and the model has not been evaluated under live UAV deployment, changing altitude, motion blur or edge-device latency. The selected 1280-pixel detector should therefore be interpreted as an accuracy-focused small-object detection result rather than as a completed UAV edge-deployment configuration. Future work should test the detector on larger geographically diverse UAV collections, repeat the data ablation under deployment-constrained 640- and 960-pixel settings, calibrate confidence thresholds for operational recall, and benchmark compressed or lower-resolution variants on UAV-class embedded AI hardware. A further translational step would be to test the dataset or detector with an environmental-monitoring partner, or within a dedicated drone-based collection study, to evaluate whether detections improve real-world recovery of hazardous vape litter.
