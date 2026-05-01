day - 1

## AI Development Lifecycle(ADLC)

### Definition:

The AI Development Lifecycle (ADLC) is the end-to-end, iterative process used to plan, build, train, deploy, and maintain artificial intelligence systems. Because an AI model learns its logic from data rather than following explicitly coded rules, the ADLC is inherently experimental and cyclical—teams frequently loop back to collect more data or retrain models when real-world performance drifts.

The Core Phases

| Phase                  | What Happens                                      | AI-Specific Twist                                                                 |
|------------------------|--------------------------------------------------|-----------------------------------------------------------------------------------|
| 1. Problem Definition  | Define the business goal, success metrics, and feasibility. | You must decide if the problem is even solvable with existing data before writing code. |
| 2. Data Engineering    | Collect, clean, label, and augment raw data.     | “Garbage in, garbage out”—this phase often consumes 60%-80% of the total time.    |
| 3. Model Development   | Select architectures and run experiments offline.| Highly exploratory; dozens of model variants may be tested and discarded.         |
| 4. Training & Validation | Feed data to the model and tune hyperparameters. | Requires significant compute (GPUs/TPUs) and careful checks for overfitting.      |
| 5. Evaluation & Testing | Test on held-out data; check for bias, fairness, and edge cases. | A model can score 99% accuracy on paper yet fail on critical minority cases.      |
| 6. Deployment & Integration | Release the model into production software.  | Often involves edge optimization, A/B testing, or fallback logic for low-confidence predictions. |
| 7. Monitoring & Maintenance | Watch for data drift, concept drift, and decaying accuracy. | Models age like milk, not wine—they must be retrained as the world changes.       |

### Example:

Imagine a startup building CropDoc, a mobile app that lets farmers snap a photo of a tomato leaf to instantly detect disease.

```
1. Problem Definition
Goal: Reduce crop loss by identifying diseases early.
Success: >95% top-1 accuracy, inference under 2 seconds on a mid-range phone, offline capability for rural fields.

2. Data Engineering
The team collects 50,000 leaf images from greenhouses. Agronomists label each image as Healthy, Early Blight, or Bacterial Spot. The team discovers the dataset is imbalanced—only 3% are Early Blight—so they use data augmentation (rotating/flipping images) to balance the classes and run cleaning scripts to remove blurry photos.

3. Model Development
Data scientists experiment with pre-trained computer vision architectures like ResNet50 and EfficientNet. They establish a simple baseline first (e.g., a small custom CNN) to ensure the problem is learnable before investing heavy compute.

4. Training & Validation
The team trains the best candidates on cloud GPUs, tracking loss curves and validation accuracy. Hyperparameters like the learning rate (η) are tuned automatically using a small search grid. They watch for overfitting: if training accuracy hits 99% but validation accuracy stalls at 88%, the model is memorizing, not learning.

5. Evaluation & Testing
The top model is tested on a strictly held-out “farmer’s field” dataset it has never seen. A confusion matrix reveals the model often confuses Early Blight with Bacterial Spot. The team adds more labeled examples of those two classes and retrains.

6. Deployment & Integration
Instead of a massive server model, the team converts the final model to TensorFlow Lite to run directly on the phone. The app is integrated so that if the model’s confidence score is below 70%, it advises the farmer to consult a human expert rather than guessing.

7. Monitoring & Maintenance
After launch, the team monitors uploads. Six months later, they notice accuracy dropping: users have started taking photos at night with flash, causing glare the original training data lacked (data drift). The team collects new nighttime images, labels them, and triggers a new ADLC loop to retrain and redeploy.
```

---