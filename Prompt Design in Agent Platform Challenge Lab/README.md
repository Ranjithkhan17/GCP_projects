# 🎨 Prompt Design in Agent Platform - Challenge Lab

> Master the art of prompt engineering with Google Cloud's Agent Platform and Gemini AI

---

## 📚 Overview

This challenge lab guides you through designing, implementing, and refining prompts using Google Cloud's Agent Platform. You'll work with Gemini to create powerful AI-driven applications for text and image analysis.

### 🎯 Learning Objectives
- Build effective prompts in Agent Studio
- Implement image analysis with Gemini
- Create customizable tagline generators
- Experiment with prompts in Python notebooks
- Fine-tune parameters for optimal results

---

## 📋 Tasks

### ✅ Task 1: Build a Gemini Image Analysis Prompt

**Objective:** Create a prompt to analyze Cymbal Direct's product images and generate descriptive text.

**What You'll Learn:**
- Creating prompts in Agent Studio
- Working with the specified model
- Generating multiple content variations

**Deliverables:**
- 📸 Short, descriptive text inspired by the image
- 💡 Catchy advertisement phrases
- 🌿 Poetic descriptions for nature-focused campaigns

**Steps:**
1. Open Agent Studio and create a new prompt
2. Configure the model_name model
3. Design your prompt to analyze product images from Google Cloud Storage
4. Experiment with different prompt variations
5. Fine-tune parameters for better results
6. **Save as:** `Cymbal Product Analysis`
7. Select your Region and save

---

### ✅ Task 2: Build a Gemini Tagline Generator

**Objective:** Create a customizable tagline generator for Cymbal Direct's outdoor product line.

**System Instructions:**
```
Cymbal Direct is partnering with an outdoor gear retailer. They're launching a new line 
of products designed to encourage young people to explore the outdoors. 
Help them create catchy taglines for this product line.
```

**Key Features:**
- 🎯 Customizable based on product attributes (durable, lightweight, etc.)
- 👥 Target audience selection (young adventurers, families, etc.)
- 💭 Emotional resonance options (empowered, connected, etc.)

**Example Input/Output:**
```
Input: Write a tagline for a durable backpack designed for hikers that makes them 
feel prepared. Consider styles like minimalist.

Output: Built for the Journey: Your Adventure Essentials.
```

**Steps:**
1. Create a new prompt in Agent Studio
2. Add system instructions (see above)
3. Include at least 2 examples to guide output
4. Add parameters for customization
5. Test with different parameter combinations
6. **Save as:** `Cymbal Tagline Generator Template`
7. Ensure autosave captures your work

---

### ✅ Task 3: Experiment with Image Analysis Code

**Objective:** Export your prompt to Python and modify it for enhanced creativity.

**What You'll Do:**
1. Navigate to **Notebooks > Workbench** in Google Cloud Console
2. Open your Workbench instance and launch **JupyterLab**
3. Open the notebook: `image-analysis.ipynb`
4. Set kernel to **Python 3 (Local)**
5. Run all cells to verify setup

**Code Exploration:**
1. Go back to Agent Studio → Cymbal Product Analysis prompt
2. Click **Code** and select **Python**
3. Copy the second code cell into your notebook
4. Run the code and verify output

**Modifications:**
1. Find the prompt text between triple quotes (`"""`)
2. Modify the prompt to produce descriptions **less than 10 words**
3. Adjust parameters to encourage **creative and unexpected** descriptions
4. 💡 **Hint:** You'll need to adjust one parameter to achieve this
5. Save and rerun the cell
6. Verify the output is shorter and more creative

---

### ✅ Task 4: Experiment with Tagline Generation Code

**Objective:** Export your tagline generator to Python and add keyword requirements.

**Code Exploration:**
1. Open the notebook: `tagline-generator.ipynb` in JupyterLab
2. Set kernel to **Python 3 (Local)**
3. Go to Agent Studio → Cymbal Tagline Generator Template
4. Click **Code** and select **Python**
5. Copy the second code cell into your notebook
6. Run and verify successful execution

**Modifications:**
1. Find the prompt text between triple quotes (`"""`)
2. Modify the last input to request taglines include the keyword **"nature"**
3. Save your changes
4. Rerun the code cell
5. Verify that the generated tagline includes the word "nature"

---

## 🚀 Getting Started

### Prerequisites
- Access to Google Cloud Console
- Agent Platform enabled in your project
- Workbench instance created
- Basic Python knowledge for notebook tasks

### Resources
- 🔗 [Agent Platform Documentation](https://cloud.google.com/docs)
- 🔗 [Gemini API Guide](https://cloud.google.com/docs/gemini)
- 🔗 [Prompt Design Best Practices](https://cloud.google.com/docs)

---

## 💡 Tips for Success

✨ **Prompt Engineering Tips:**
- Start simple and iterate
- Use examples to guide model behavior
- Experiment with temperature and other parameters
- Test with different input variations
- Document what works and what doesn't

🔍 **Debugging:**
- Check model output format
- Verify image paths in Cloud Storage
- Review error messages in notebooks
- Use print statements to debug parameters

---

## 📊 Success Criteria

- ✅ All 4 tasks completed
- ✅ Prompts saved with correct names
- ✅ Python code modifications working
- ✅ Output meets specified requirements
- ✅ Creative and optimized results achieved

---

## 📝 Notes

- Save your work regularly using autosave
- Test prompts with multiple inputs
- Document your parameter adjustments
- Keep notebook outputs for reference

---

**Happy Prompting! 🎉** Build amazing AI applications with Gemini and Agent Platform.
