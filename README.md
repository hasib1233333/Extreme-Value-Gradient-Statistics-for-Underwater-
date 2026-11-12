Title:

Extreme Value Gradient Statistics for Underwater Turbidity Estimation and Their Validation in Real Water

Abstract:

Few underwater vehicles carry a turbidity meter, yet nearly every vehicle carries a camera. We study turbidity cues computed from the extreme values of locally normalised image gradients and summarised by a closed-form generalised extreme value fit. The resulting contrast index is exactly invariant to a common camera gain. Under stated assumptions it separates into scene, forward-scatter and veiling-light terms, and the ratio of the fitted locations at two scales gives the width of blurred edges. This inversion fails for about half of natural images, which are then flagged as invalid. On 1315 tank images and 1080 held-out fish images with turbidity measured in nephelometric turbidity units, the training-free index follows turbidity closely when only the water changes (Spearman rank correlation 8.10 × 10⁻¹) but not when the scene changes. With three calibrated coefficients the cues reach 8.36 × 10⁻¹ on the tank data and 5.38 × 10⁻¹ on an untouched hold-out split, among the leading methods but below a pair of simple contrast statistics, and they give the lowest median turbidity error when transferred from the tank to the fish images. A Type A uncertainty evaluation attributes 92% of the error variance to the scene and agrees with distribution-free prediction intervals. Upper percentiles of the same gradients outperform the fitted location, so the cues are best used on underwater vehicles as coarse indicators of water and image quality. Attenuation estimates are validated only in simulation.

Keywords:

Underwater imaging, Turbidity, Extreme value theory, Autonomous underwater vehicles, Measurement uncertainty
