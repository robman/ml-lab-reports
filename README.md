# **ml-lab-reports - Because ML Dev is NOT Software Dev**

![ML Lab Reports - An "Experiment-focused" Git Workflow for ML Development](/img/ml-lab-reports-an-experiment-focused-approach-to-ML.png)

**Machine Learning Development: An "Experiment-focused" Git Workflow to make ML Dev more Reproducible**

Machine Learning (ML) development is an amazing journey of experimentation, iteration, and discovery. Unlike traditional software development, where code evolves incrementally towards a stable release, ML thrives on continuous experimentation to fine-tune models, tweak features, and evaluate outcomes. Managing this experimental nature poses unique challenges, especially when it comes to tracking progress, ensuring reproducibility, and facilitating collaboration.

A lot of time can also be spent trying to replicate work that others have published to verify baselines and explore ideas. However, even when they publish their code on Github it can be very time consuming to deal with specific dependencies and the constantly moving APIs of the ML tools. Reproducing their work is neither straightforwards or easy. It's amazing just how fragile the dependency structure in python and the various ML tools can really be.

In this tutorial, I’ll walk you through a streamlined workflow that leverages Git not just for version control, but as a robust system for managing ML experiments. By treating each Git branch as a standalone "lab report", you can maintain organised, reproducible experiments while seamlessly integrating environment management. Let’s dive in!

---

## **Why ML Development Differs from Traditional Software Development**

Before delving into the workflow, it’s essential to understand the fundamental differences between ML development and conventional software engineering:

1. **Experimental Nature:**
   - **ML Development:** Involves continuous experimentation with data, features, models, and hyperparameters to achieve optimal precision or performance.
   - **Software Development:** Focuses on building and refining features to create a stable, functional product.

2. **Reproducibility:**
   - **ML Development:** Reproducibility is critical to validate results, compare experiments, and ensure consistency across different environments.
   - **Software Development:** While important, the emphasis is more on code stability and maintainability.

3. **Data Dependency:**
   - **ML Development:** Relies heavily on data quality and preprocessing steps, making version control and data management paramount.
   - **Software Development:** Less dependent on external data sources, focusing more on codebase management.

Understanding these distinctions highlights the necessity for specialised workflows tailored to the unique demands of ML projects.

---

## **Goals of the New Workflow**

The primary objectives in setting up this Git-based workflow are:

1. **Enhanced Experiment Management:**
   - Organise each experiment as a separate Git branch, encapsulating all relevant code, data, and results.

2. **Reproducibility:**
   - Ensure that every experiment can be replicated precisely, including the environment setup.

3. **Seamless Environment Management:**
   - Incorporate Conda environments for each experiment, utilising `environment.yml` and `requirements.txt` for dependency management.

4. **Simplicity and Standalone Operation:**
   - Maintain a workflow that operates independently of external tools like MLflow, leveraging existing tools on your device or dev environment..

5. **Ease of Collaboration:**
   - Facilitate sharing and collaborating on experiments in a reproducible manner.

---

## **Introducing a Git-based Experiment-focused Workflow**

### **1. Repository Initialisation and Setup**

Start by initialising your project directory as a Git repository:

```bash
# Initialise Git repository
git init -b wip my-ml-project
cd my-ml-project

# Optional: Create a .gitignore file to exclude unnecessary files
echo "__pycache__/
*.pyc
.env
*.sw*
" > .gitignore
git add .gitignore
git commit -m "Initial commit with .gitignore"
```

### **2. Conduct an Experiment**

In this workflow, each experiment is encapsulated within its own Git branch, serving as a comprehensive "lab report". This branch contains all necessary components to replicate the experiment, including code, data, logs, and documentation.

You are free to create any data, files and code you want without having to individually add these files to this Git repository. Just make sure you're on the `wip` branch and that this branch has nothing significant committed. This is your primary workspace.

### **3. Automating Lab Report Creation with a Bash Script**

To streamline the creation of these branches, a custom bash script named `ml_lab_report` is employed. This script automates the process of stashing current work, creating a new branch, pop'ing stashed changes, and committing the experiment details. Then it returns all your work to the state it was before you created your latest `ml_lab_report`, so you can seamlessly carry on from where you were without worrying if files are staged or committed or not. 

**`ml_lab_report` Script:**

```bash
#!/bin/bash

# Process title
if [ -z "$1" ]
then
  echo "A lab report title is required"
  exit 1
fi

# Generate a timestamp
timestamp=$(date +"%Y-%m-%d-%H%M%S")

# Prepare branch name
title="$1"
branch_name="${timestamp}-${title// /-}"

# Ensure the current directory is a Git repository
git rev-parse --is-inside-work-tree >/dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "${PWD} is not a Git repository."
  exit 1
fi

# Stash any uncommitted changes
git stash -u -m "WIP: $title"

# Create and switch to a new branch
git checkout -b "$branch_name"

# Apply stashed changes but keep the stash so we can restore it on the wip branch
git stash apply 

# Stage all changes
git add -A .

# Commit with a template
git commit -a -t ~/.vim/templates/commit-ml-lab-report.txt

# Switch back to wip
git checkout wip

# Put all your work back to how it was
git stash pop

echo "Created lab report branch: $branch_name"
```

**Key Features:**

- **Timestamped Branch Names:** Ensures unique and chronological branch naming.
- **Stashing Mechanism:** Safeguards uncommitted work before branching.
- **Commit Templates:** Facilitates structured documentation for each experiment.
- **Seamless:** Returns your wip workspace back to exactly how it was before you created your latest lab report. 

**Setting Up Commit Templates:**

Create a commit template file at `~/.vim/templates/commit-ml-lab-report.txt`:

```markdown
# One-line title should be auto-added above (if not, add one)

intro: 

goals:

method:

results:
```

### **4. Implementing Git Hooks for Structured Commit Messages**

Git hooks automate tasks during Git operations. Here, a `prepare-commit-msg` hook is used to prepend structured sections to commit messages, ensuring consistency across lab reports.

**`prepare-commit-msg` Hook:**

```bash
#!/bin/sh

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2
SHA1=$3

# Add title and structured sections
echo "title: $title\n\n$(cat $COMMIT_MSG_FILE)" > "$COMMIT_MSG_FILE"
```

**Setting Up the Hook:**

1. Save the above script as `.git/hooks/prepare-commit-msg`.
2. Make it executable:

   ```bash
   chmod +x .git/hooks/prepare-commit-msg
   ```

---

## **Incorporating Conda Environments for Reproducibility**

Managing dependencies is crucial for replicating experiments. Often, a specific experiment will require specific tools or versions of tools. Utilising Conda environments provides isolated and consistent environments per experiment.

### **1. Setting Up a Conda Environment for an Experiment**

**a. Creating the Environment:**

After setting up your base project, create a Conda environment tailored for your experiment:

```bash
conda create -n experiment-env python
conda activate experiment-env
```

**b. Installing Dependencies:**

Install necessary packages using `pip` or `conda`:

```bash
# Example with pip
pip install tensorflow keras numpy pandas matplotlib

# Or using conda
conda install tensorflow keras numpy pandas matplotlib
```

### **2. Exporting Dependencies**

To ensure others (or future you) can recreate the environment, export the dependencies to `environment.yml` and `requirements.txt`.

**a. Export `environment.yml`:**

```bash
conda env export --no-builds > environment.yml
```

**b. Export `requirements.txt`:**

This is particularly useful if some dependencies were installed via `pip`.

```bash
pip freeze > requirements.txt
```

These files will now be automatically included in your `ml_lab_report`. You may like to add these lines near the top of your `ml_lab_report` bash script so you always include the most updated versions of these files. I like to include them just before the `git stash -u ...` line so they are then included in your wip workspace when that is restored.

Just remember to activate the right conda env before you try to replicate any experiment, or set one up correctly before you start a new experiment. Again, this is where the `ml_lab_report` comes in because it documents this for you, and you can add extra descriptions in the `method` section if required.

---

## **Step-by-Step Guide: Setting Up the Workflow**

Here’s a comprehensive guide to setting up this Git-based ML experiment-focused workflow on your local machine.

### **1. Prerequisites**

- **Git:** Ensure Git is installed. [Download Git](https://git-scm.com/downloads)
- **Conda:** Install Anaconda or Miniconda. [Download Conda](https://docs.conda.io/en/latest/miniconda.html)
- **Bash:** Unix-like shell environment.
- **Text Editor:** Vim or your preferred editor for creating scripts.

### **2. Initialise the Git Repository**

```bash
mkdir my-ml-project
cd my-ml-project
git init -b wip
```

### **3. Create the `.gitignore` File**

```bash
echo "__pycache__/
*.pyc
.env
*.sw*
" > .gitignore
git add .gitignore
git commit -m "Initial commit with .gitignore"
```

### **4. Develop the `ml_lab_report` Script**

Create a new file named `ml_lab_report`. I like to store this in my personal `bin/` directory so it's always in my `$PATH`. But you may just like to add an alias or perhaps even include it in your wip workspace.

```bash
vim ml_lab_report
```

**Paste the following content:**

```bash
#!/bin/bash

# Process title
if [ -z "$1" ]
then
  echo "A lab report title is required"
  exit 1
fi

# Generate a timestamp
timestamp=$(date +"%Y-%m-%d-%H%M%S")

# Prepare branch name
title="$1"
branch_name="${timestamp}-${title// /-}"

# Ensure the current directory is a Git repository
git rev-parse --is-inside-work-tree >/dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "${PWD} is not a Git repository."
  exit 1
fi

# Get the latest env config
conda env export --no-builds > environment.yml
pip freeze > requirements.txt

# Stash any uncommitted changes
git stash -u -m "WIP: $title"

# Create and switch to a new branch
git checkout -b "$branch_name"

# Apply stashed changes but keep the stash so we can restore it on the wip branch
git stash apply 

# Stage all changes
git add -A .

# Commit with a template
git commit -a -t ~/.vim/templates/commit-ml-lab-report.txt

# Switch back to wip
git checkout wip

# Put all your work back to how it was
git stash pop

echo "Created lab report branch: $branch_name"
```

**Make the script executable:**

```bash
chmod +x ml_lab_report
```

### **5. Set Up the Commit Template**

Create the commit message template at `~/.vim/templates/commit-ml-lab-report.txt`, or wherever you prefer:

```bash
mkdir -p ~/.vim/templates
vim ~/.vim/templates/commit-ml-lab-report.txt
```

**Paste the following content:**

```markdown
# One-line title should be auto added above (if not, add one)

intro: 

goals:

method:

results:
```

### **6. Implement the Git Hook**

Create the `prepare-commit-msg` hook in your repository:

```bash
vim .git/hooks/prepare-commit-msg
```

**Paste the following content:**

```bash
#!/bin/sh

COMMIT_MSG_FILE=$1
COMMIT_SOURCE=$2
SHA1=$3

echo "title: $title\n\n$(cat $COMMIT_MSG_FILE)" > "$COMMIT_MSG_FILE"
```

**Make the hook executable:**

```bash
chmod +x .git/hooks/prepare-commit-msg
```

### **7. Create the Conda Environment and Export Dependencies**

**a. Create and Activate the Environment:**

```bash
conda create -n experiment-env python
conda activate experiment-env
```

**b. Install Dependencies:**

```bash
pip install tensorflow keras numpy pandas matplotlib
```

---

## **End-to-End Example: A Simple TensorFlow/Keras Experiment**

To illustrate the workflow, let’s walk through a simple project with your first experiment: training a basic neural network on the MNIST dataset.

### **1. Setup for the Experiment**

**a. Create your env and working directory:**

You may like to create a script that does this all in one shot for you.

```bash
project='mnist-demo'
conda create -n $project python
conda activate $project 
pip install tensorflow keras numpy pandas matplotlib
git init -b wip $project 
cd $project 
cp $YOUR_TEMPLATES_DIR/ml-lab-report-prepare-commit-msg .git/hooks/prepare-commit-msg
chmod +x .git/hooks/prepare-commit-msg
```

HINT: If you create this as a script then you'll need to add a `. ` before it when you run it so the `cd` also udpates your current working directory.

*This creates a new working environment and a git repos in which you can conduct your experiment and create `ml_lab_reports`. 

### **2. Developing the Experiment**

**a. Create the Training Script: `train.py`**

```python
import tensorflow as tf
from tensorflow.keras.datasets import mnist
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Flatten
from tensorflow.keras.utils import to_categorical

# Load and preprocess data
(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0
y_train, y_test = to_categorical(y_train), to_categorical(y_test)

# Define the model
model = Sequential([
  Flatten(input_shape=(28, 28)),
  Dense(128, activation='relu'),
  Dense(10, activation='softmax')
])

# Compile the model
model.compile(
  optimizer='adam',
  loss='categorical_crossentropy',
  metrics=['accuracy']
)

# Train the model
model.fit(x_train, y_train, epochs=5, batch_size=32, validation_split=0.1)

# Evaluate the model
test_loss, test_acc = model.evaluate(x_test, y_test)
print(f'Test accuracy: {test_acc}')
```

**b. Run: `python train.py > train.log 2>&1`**

Run your experiment and gather the output into `train.log` or some consistent place you use for each experiment. Once you believe your experiment is complete then collect the key results to add to your lab report in the next step. If I'm running multiple attempts for this one experiment then I create a `results/` directory and create an attempt number inside there for each version I run e.g. `results/01`. Then I copy the version of `train.py` I used and the `train.log` it generated, plus any evaluation output (e.g. confusion matrix images, etc.) into that results dir. Then I have a completely reproducible bundle for each attempt within this experiment.

### **3. Documenting the Experiment**

**a. Create your `ml_lab_report`:**

Run your `ml_lab_report` script with a name for your experiment. The project dir and conda env already have the project name (e.g. 'mnist-demo'), so you only need to choose a specific title that describes your current experiment. Remember, this is automatically prefixed with a timestamp when it's turned into a branch name so even if you run similarly named experiments multiple times you'll still be able to see where they fit in chronological order.

```bash
ml_lab_report 'My first basic attempt'
```

**Inside your configured git editor, you’ll see:**

```markdown
title: My first basic attempt 

intro: 

goals:

method:

results:
```

**Fill in the sections:**

```markdown
title: My first basic attempt 

intro: 
This experiment focuses on training a simple neural network using TensorFlow and Keras on the MNIST dataset to classify handwritten digits.

goals:
- Implement a basic neural network model.
- Achieve at least 90% accuracy on the test dataset.
- Document the training process and results.

method:
- Loaded and preprocessed the MNIST dataset.
- Defined a Sequential model with Flatten and Dense layers.
- Compiled the model with Adam optimiser and categorical crossentropy loss.
- Trained the model for 5 epochs with a batch size of 32.
- Evaluated the model on the test dataset.

results:
- The model achieved a test accuracy of 97.5%, surpassing the target of 90%.
- Training and validation accuracies consistently improved over epochs without signs of overfitting.
```

*Save and exit the editor to finalise the commit.*

### **4. Reviewing and Sharing the Experiment**

With the experiment encapsulated in its own branch, it’s easy to review, replicate, or share. And at any time you can review your entire history of `ml_lab_reports` for this project by running `git log --all`.

---

## **Adapting the Workflow to Popular IDEs**

Modern Integrated Development Environments (IDEs) like **Visual Studio Code (VS Code)** and **PyCharm** offer robust Git integration, making it seamless to adopt this workflow.

### **1. Visual Studio Code (VS Code)**

- **Built-in Git Support:** Easily manage branches, commits, and stashes through the source control panel.
- **Integrated Terminal:** Run your `ml_lab_report` script directly within VS Code’s terminal.
- **Extensions:**
  - **GitLens:** Enhances Git capabilities with visualisations and insights.
  - **Python Extension:** Provides robust support for Python development, including Conda environment management.

**Setup Steps:**

1. **Open the Project:**
   - Launch VS Code and open your ML project directory.

2. **Access Git Features:**
   - Use the source control panel to create, switch, and manage branches.

3. **Run the `ml_lab_report` Script:**
   - Open the integrated terminal (`Ctrl + \``) and execute:
     ```bash
     ./ml_lab_report "New Experiment Title"
     ```

4. **Manage Conda Environments:**
   - Use the Python extension to select the appropriate Conda environment based on `environment.yml`.

### **2. PyCharm**

- **Comprehensive Git Integration:** Visual tools for managing branches, commits, and diffs.
- **Conda Support:** Easily configure and switch between Conda environments.
- **Terminal Access:** Execute scripts without leaving the IDE.

**Setup Steps:**

1. **Open the Project:**
   - Launch PyCharm and open your ML project directory.

2. **Configure Git:**
   - Ensure Git is properly configured in `Settings > Version Control > Git`.

3. **Run the `ml_lab_report` Script:**
   - Open the terminal within PyCharm and execute:
     ```bash
     ./ml_lab_report "New Experiment Title"
     ```

4. **Manage Conda Environments:**
   - Navigate to `Settings > Project > Python Interpreter` to select or update the Conda environment based on `environment.yml`.

---

## **Bundling and Sharing Experiments for Collaboration**

One of the standout benefits of this workflow is the ease of sharing and collaborating on experiments. By encapsulating each experiment within its own Git branch, collaborators can:

- **Review Experiments Independently:**
  - Each branch serves as an isolated lab report, making it straightforward to examine specific experiments without cluttering the main codebase.

- **Replicate Experiments with Ease:**
  - With `environment.yml` and `requirements.txt` included, reproducing the exact environment is hassle-free.

- **Compare Different Approaches:**
  - Utilise Git’s diff and merge capabilities to compare different branches, strategies, and results.

**Sharing Steps:**

1. **Push the Branch to a Remote Repository:**

   ```bash
   git push origin "timestamp-Experiment-Title"
   ```

2. **Collaborators Clone and Checkout:**

   ```bash
   git clone https://github.com/yourusername/my-ml-project.git
   cd my-ml-project
   git checkout "timestamp-Experiment-Title"
   ```

3. **Set Up the Conda Environment:**

   ```bash
   conda env create -f environment.yml
   conda activate experiment-env
   ```

*Collaborators can now run, modify, and further experiment with the shared branch seamlessly.*

---

## **Key Benefits of a Git-based Experiment-focused Workflow**

1. **Reproducibility:**
   - Ensures that every experiment can be replicated precisely, preserving code, data, dependencies, and results.

2. **Organisation:**
   - Keeps experiments neatly organised within separate branches, avoiding clutter in the main codebase.

3. **Collaboration:**
   - Facilitates sharing and collaborative development by encapsulating experiments within Git branches.

4. **Automation:**
   - Streamlines experiment setup through automation scripts and Git hooks, reducing manual overhead.

5. **Flexibility:**
   - Integrates seamlessly with various tools and IDEs, adapting to different workflows and preferences.

6. **Documentation:**
   - Encourages thorough documentation of experiments through structured commit messages, enhancing knowledge sharing.

7. **Version Control:**
   - Leverages Git’s robust version control features to track changes, experiment iterations, and progress over time.

---

## **Conclusion**

Machine Learning development thrives on experimentation and iteration, demanding workflows that can keep pace with rapid changes and complex dependencies. By repurposing Git branches as comprehensive lab reports and integrating Conda environment management, this workflow not only enhances organisation and reproducibility but also fosters collaboration and ease of sharing.

While tools like **MLflow** offer powerful features for experiment tracking, adopting a standalone Git-based approach provides a lightweight and flexible alternative that seamlessly integrates with existing tools and environments. Whether you’re working solo or as part of a team, this workflow empowers you to manage your ML experiments with precision, clarity, and efficiency.

---

**Feel free to connect and share your experiences or ask questions about setting up your ML workflows. Let’s advance together in building robust and reproducible machine learning models!**

---

**#MachineLearning #Git #Workflow #Reproducibility #DataScience #Conda #TensorFlow #Keras #SoftwareDevelopment #ExperimentManagement**
