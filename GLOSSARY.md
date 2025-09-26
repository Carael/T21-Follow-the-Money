# Fraud Detection Challenge - Glossary

## 📚 **Core Concepts**

### **P2P (Peer-to-Peer) Payments**
Digital money transfers between individuals, like Venmo, PayPal, or Zelle. Users can send money directly to each other without traditional bank intermediaries.
- **Example**: John sends $50 to Sarah for splitting dinner
- **Challenge**: Easy for fraudsters to exploit for money laundering

### **Embeddings**
Dense, low-dimensional vector representations of data that capture semantic relationships and patterns.
- **Purpose**: Convert categorical data (account IDs) into numerical vectors
- **Example**: Account "AC1234" → `[0.2, -0.7, 1.1, 0.4, ..., -0.3]` (32 numbers)
- **Benefit**: Accounts with similar behavior get similar vectors

### **Semantic Similarity**
The concept that similar entities should have similar representations in the embedding space.
- **Example**: Two money mules should have similar embeddings because they exhibit similar network patterns
- **Application**: Used to detect fraud rings and suspicious account clusters

---

## 🚨 **Fraud Detection Terms**

### **Class Imbalance**
When one class (fraud) is much rarer than another (legitimate transactions).
- **Our dataset**: ~1.6% fraud vs 98.4% legitimate
- **Challenge**: Model tends to predict "not fraud" for everything
- **Solution**: Specialized metrics like PR-AUC instead of accuracy

### **Label Noise**
Imperfect labels that don't match ground truth, simulating real-world uncertainty.
- **False Positives**: Legitimate transactions labeled as fraud (170 cases in our data)
- **False Negatives**: Fraud transactions labeled as legitimate (21 cases in our data)
- **Reality**: Banks don't have perfect fraud labels

### **Money Mule Networks**
Fraud scheme where criminals recruit people to receive and transfer stolen money.
- **Pattern**: Many accounts → mule account → boss account
- **Detection**: "Gathering then scattering" transaction patterns
- **Graph signature**: High in-degree followed by high out-degree

### **Account Takeover**
When fraudsters gain access to legitimate accounts and change behavior patterns.
- **Signals**: Sudden changes in transaction amounts, timing, or recipients
- **Detection**: Compare current behavior to historical patterns

---

## 📊 **Machine Learning Metrics**

### **PR-AUC (Precision-Recall Area Under Curve)**
Primary metric for imbalanced classification, focusing on how well we identify the rare positive class.
- **Better than ROC-AUC**: Not fooled by class imbalance
- **Precision**: Of flagged transactions, how many are actually fraud?
- **Recall**: Of actual fraud, how many do we catch?

### **Recall@0.5% FPR**
Operational metric: percentage of frauds caught while keeping false positive rate ≤ 0.5%.
- **Business constraint**: Banks can only investigate limited transactions
- **0.5% FPR**: Flag max 5 out of 1000 legitimate transactions
- **Trade-off**: Lower FPR = fewer false alarms but might miss some fraud

### **False Positive Rate (FPR)**
Percentage of legitimate transactions incorrectly flagged as fraud.
- **Cost**: Each false positive requires human investigation
- **Goal**: Minimize while still catching real fraud

### **Temporal Leakage**
When model accidentally learns from "future" data it shouldn't have access to.
- **Problem**: Unrealistically optimistic performance
- **Prevention**: Time-based train/validation splits
- **Example**: Training on March-May data, testing on June data

---

## 🕸️ **Graph Theory & Network Analysis**

### **Graph/Network**
Mathematical structure representing relationships between entities.
- **Nodes**: Accounts (e.g., AC0053, AC1926)
- **Edges**: Transactions between accounts
- **Weights**: Transaction amounts or frequencies

### **Node2Vec/DeepWalk**
Algorithms that learn node embeddings by performing random walks on graphs.
- **Process**: Walk through network like browsing social media connections
- **Training**: Use walks as "sentences" to train Word2Vec-style model
- **Result**: Nodes with similar neighborhoods get similar embeddings

### **Random Walk**
Sequence of steps through a graph where each step chooses next node randomly.
- **Example**: AC0053 → AC1926 → AC2284 → AC0751 → ...
- **Purpose**: Capture local neighborhood structure around each node
- **Analogy**: Like wandering through a social network by clicking on friends

### **Cosine Similarity**
Measure of similarity between two vectors based on angle between them.
- **Range**: -1 (opposite) to +1 (identical)
- **Use**: Compare account embeddings to find similar behavior patterns
- **Formula**: `cos(θ) = (A·B) / (|A|×|B|)`

---

## 🔧 **Feature Engineering**

### **Temporal Features**
Features that capture time-based patterns and changes.
- **Velocity**: Time since last transaction
- **Recency**: How recently account was active
- **Frequency**: Transaction rate over time windows

### **Activity Features (Cumulative)**
Running statistics that accumulate over time using only past data.
- **tx_count**: Number of transactions sent/received so far
- **unique_partners**: Number of different accounts interacted with
- **total_amt**: Cumulative amount sent/received

### **Pair Familiarity**
Features measuring relationship strength between account pairs.
- **Count**: How many times this sender-receiver pair has transacted
- **Amount**: Total amount exchanged between this pair
- **Recency**: Time since last transaction between this pair

### **Z-Score Normalization**
Statistical technique to standardize features by removing mean and scaling by standard deviation.
- **Formula**: `(value - mean) / std_dev`
- **Purpose**: Compare values relative to typical behavior
- **Anti-leakage**: Use only training data statistics

### **Log1p Transformation**
Logarithmic transformation that handles zero values: `log(1 + x)`
- **Purpose**: Reduce skewness in transaction amounts
- **Benefit**: Makes highly skewed distributions more normal
- **Handles zeros**: Unlike log(x), log1p(0) = 0

---

## 🎯 **Validation & Evaluation**

### **Time-Based Split**
Splitting data chronologically rather than randomly to prevent temporal leakage.
- **Training**: March 1 - May 14 (56K transactions)
- **Validation**: May 14 - May 31 (14K transactions)
- **Principle**: Train on past, test on future

### **Fit-Transform Pattern**
ML best practice to prevent data leakage in preprocessing.
- **Fit**: Learn statistics (mean, std) from training data only
- **Transform**: Apply same statistics to both training and validation
- **Never**: Calculate statistics on combined data

### **Ablation Study**
Systematic comparison of model variants to measure component contributions.
- **Baseline**: Tabular features only
- **Enhanced**: Tabular + graph embeddings
- **Goal**: Quantify value added by graph features

---

## 🏗️ **Technical Implementation**

### **Anti-Leakage Framework**
Code structure ensuring no future information contaminates training features.
- **Temporal ordering**: All features use only past transactions
- **Split validation**: Strict time-based separation
- **Feature computation**: Rolling/cumulative calculations with time constraints

### **Contrastive Learning**
Training method that learns by comparing positive and negative examples.
- **Anchor**: Reference transaction window
- **Positive**: Similar/related transaction window
- **Negative**: Dissimilar transaction window
- **Goal**: Make similar items closer, dissimilar items farther apart

### **Graph Embedding Pipeline**
End-to-end process for generating network-based features.
1. **Graph Construction**: Build network from transaction data
2. **Walk Generation**: Create random walks through network
3. **Embedding Training**: Learn vector representations
4. **Feature Creation**: Generate transaction-level features from embeddings

---

## 🎪 **Hackathon Strategy**

### **Tabular + Graph-Lite**
Hybrid approach combining traditional ML with lightweight graph methods.
- **Tabular**: Fast, reliable baseline features
- **Graph-Lite**: Simple embeddings to capture network effects
- **Time-boxed**: Each phase has strict time limits for hackathon pace

### **Sprint Methodology**
Structured approach with measurable milestones and quick iteration.
- **Phase goals**: Clear objectives and deliverables
- **Success metrics**: Quantitative targets defined upfront
- **Scope control**: Prevent feature creep and endless optimization

### **Incremental Value**
Demonstrating that each additional component improves performance.
- **Baseline measurement**: Establish performance floor
- **Component addition**: Add one feature type at a time
- **Value quantification**: Measure exact improvement from each addition

---

## 💼 **Business Context**

### **Operational Constraints**
Real-world limitations that affect model design and evaluation.
- **Investigation capacity**: Limited human reviewers for flagged transactions
- **False positive costs**: Time and resources wasted on false alarms
- **False negative costs**: Actual fraud losses and regulatory penalties

### **Fraud Evolution**
Dynamic nature of fraud requiring adaptable detection systems.
- **Adversarial environment**: Criminals adapt to detection methods
- **New typologies**: Novel fraud schemes constantly emerging
- **Generalization challenge**: Models must work on unseen fraud patterns

### **Regulatory Requirements**
Compliance and explainability needs in financial services.
- **Model interpretability**: Must explain why transactions are flagged
- **Audit trails**: Decision processes must be documented
- **Fair lending**: Avoid discriminatory patterns in fraud detection

---

*This glossary covers all key terms from our P2P fraud detection challenge. Refer back as needed during implementation!*