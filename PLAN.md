# P2P Fraud Detection Challenge - Sprint Plan

## 🎯 Strategy Overview
**Approach**: "Tabular + Graph-Lite" Sprint  
**Timeline**: Tight hackathon execution with measurable milestones  
**Data**: 70K P2P transactions with sender/receiver, timestamp, amount, currency, and ground truth  

## 🏆 Success Criteria (Defined Upfront)
- **Beat tabular baseline** on PR-AUC and recall@low-FPR (0.5%)
- **Show incremental value** from simple graph/context features
- **Prevent scope creep** with measurable results at each phase

## 📊 Key Metrics Explained

### PR-AUC (Precision-Recall Area Under Curve)
- **Why**: Handles massive class imbalance in fraud detection
- **Better than ROC-AUC**: Focuses on rare positive cases (fraud)
- **Measures**: Trade-off between precision and recall

### Recall@0.5% FPR (False Positive Rate)  
- **Real-world constraint**: Banks can't investigate every transaction
- **0.5% FPR**: Out of 1000 legit transactions, flag only ~5 as suspicious
- **Business relevance**: Matches operational capacity for human review

## 🚀 Execution Phases

### Phase 0: Data Split & Temporal Hygiene (15-20 min)
**Critical Anti-Leakage Measures**
- [ ] Parse timestamps, sort P2P transactions by time
- [ ] **Time-based split**: Last 20% by timestamp = validation (NO shuffling!)
- [ ] **Leakage guard**: All cumulative/rolling features use only past data
- [ ] Implement proper temporal validation framework

### Phase 1: Fast Baseline Features - Tabular (30-40 min)
**Minimal features that usually move the needle:**

#### Amount Features
- [ ] Raw amount
- [ ] log1p(amount) 
- [ ] Z-score per sender/receiver (using only past data)

#### Activity/Degree Features (Cumulative)
- [ ] `sender_tx_count`, `receiver_tx_count`
- [ ] `sender_unique_partners`, `receiver_unique_partners`  
- [ ] `sender_total_amt`, `receiver_total_amt`

#### Velocity/Recency Features
- [ ] Time since sender's last transaction
- [ ] Time since last (sender,receiver) edge
- [ ] Transaction frequency patterns

#### Pair Familiarity
- [ ] Cumulative count for (sender,receiver) pair
- [ ] Cumulative amount for (sender,receiver) pair

#### Technical Requirements
- [ ] Currency one-hot encoding (or drop if single value)
- [ ] **Fit-transform pattern**: Fit on train, transform on validation
- [ ] Ensure no future data leakage in feature computation

### Phase 2: Tiny Graph "Semantics" (25-30 min)
**Capture interaction structure with lightweight embeddings:**

#### Graph Construction
- [ ] Build undirected graph: nodes = accounts
- [ ] Edge weights = transaction count OR log(sum(amount))
- [ ] Handle multiple transactions between same pair

#### Embedding Generation  
- [ ] Run Node2Vec/DeepWalk (small dims = 32)
- [ ] **Zero-dependency fallback**: NetworkX walks + gensim.Word2Vec
- [ ] Generate embeddings for all account nodes

#### Feature Creation
- [ ] Concatenate emb(sender) + emb(receiver)
- [ ] Cosine similarity between sender and receiver embeddings
- [ ] Optional: Embedding distance metrics

#### Validation
- [ ] Recompute PR-AUC and recall@FPR with graph features
- [ ] Compare against tabular baseline

### Phase 3: Ablation Study & Performance Analysis (20-30 min)
**Systematic comparison and validation:**

#### Performance Comparison
- [ ] **Baseline**: Tabular features only
- [ ] **Enhanced**: Tabular + graph embeddings  
- [ ] **Optional**: Individual component ablation

#### Metrics Calculation
- [ ] PR-AUC for both models
- [ ] Recall@0.5% FPR for both models
- [ ] Calibration curves
- [ ] Top-k precision analysis

#### Feature Analysis
- [ ] Feature importance rankings (XGBoost gain)
- [ ] Sanity check: pair_count, time_since_last, degree features should rank high
- [ ] Validate signal quality and interpretability

### Phase 4: Advanced Contrastive Embeddings (If Time Permits)
**Optional enhancement for remaining time:**

#### Contrastive Learning Setup
- [ ] **Anchor**: Sender's window of 5 transactions
- [ ] **Positive**: Later window of same sender  
- [ ] **Negatives**: Windows from other senders

#### Model Architecture
- [ ] Train tiny 2-layer MLP on aggregated transaction stats
- [ ] Learn to separate anchor-positive from anchor-negative pairs
- [ ] Extract hidden layer as user embedding

#### Integration
- [ ] Use learned embeddings as additional features
- [ ] Plug into main fraud detection model
- [ ] Measure incremental improvement

## 🧠 Why This Approach Works

### Temporal/Behavioral Features
- **Sequential dependencies**: Who pays whom, how often, when
- **Velocity patterns**: Unusual timing or frequency changes
- **Pair familiarity**: Repeated interactions vs. new connections

### Random-Walk Embeddings  
- **Neighborhood capture**: Accounts with similar payment partners
- **Structural similarity**: Similar positions in payment network
- **Context approximation**: Rich representation without billions of transactions

### Combined Power
- **Tabular ML**: Transaction-level patterns and statistics
- **Graph methods**: Network structure and relationships
- **Manageable scope**: Concrete results within hackathon timeline

## 📋 Implementation Checklist

### Environment Setup
- [ ] Python environment with pandas, scikit-learn, networkx
- [ ] Optional: node2vec, gensim for embeddings
- [ ] Jupyter notebook for iterative development

### Data Loading & Validation
- [ ] Load all CSV files and validate schema
- [ ] Check data quality and missing values
- [ ] Verify timestamp parsing and sorting

### Baseline Model
- [ ] Implement tabular feature pipeline
- [ ] Train initial model (XGBoost/RandomForest)
- [ ] Establish baseline performance metrics

### Graph Enhancement
- [ ] Build graph from transaction data  
- [ ] Generate and validate embeddings
- [ ] Integrate with tabular features

### Final Evaluation
- [ ] Compare all model variants
- [ ] Generate final performance report
- [ ] Prepare presentation materials

---

## 🎯 Expected Outcomes

**Minimum Viable Product**: Working fraud detection model with clear performance metrics  
**Success Scenario**: Graph features provide measurable improvement over tabular baseline  
**Stretch Goal**: Interpretable model with actionable insights for fraud investigators

**Time Budget**: ~6-8 hours total execution time with built-in buffers for debugging and iteration 
 
##goals (decide success up front)
beat a plain tabular baseline on PR-AUC and recall@low-FPR (e.g., 0.5%).
show incremental value from simple graph/context features.
phase 0 — split & hygiene
parse timestamps, sort by time.
time-based split: last 20% by timestamp = validation. (no shuffling!)
leakage guard: any cumulative/rolling feature must only use past data.

##phase 1 — fast baseline features
Minimal features that usually move the needle:
amount features: raw, log1p, z-score per sender/receiver (use only past).
activity/degree (cumulative):
sender_tx_count, receiver_tx_count
sender_unique_partners, receiver_unique_partners
sender_total_amt, receiver_total_amt
velocity / recency: time since sender’s last tx; time since last (sender,receiver) edge.
pair familiarity: cumulative count and cumulative amount for (sender,receiver).
currency one-hot (or drop if only one value).
Repeat the same transformations on valid with state carried from train (fit-transform pattern or recompute on concat then mask future with time <= cut).
 
#phase 2 — tiny graph “semantics” (25–30 min)
Capture the interaction structure with a light random-walk embedding:
build an undirected graph where nodes are accounts; each edge is a (sender,receiver) pair with weight = count or log sum(amount).
run Node2Vec/DeepWalk (small dims, e.g., 32). If you want zero extra deps, simulate short walks with networkx and train gensim.Word2Vec.
generate embeddings for accounts; create features by concatenating emb(sender) and emb(receiver) and/or their cosine similarity.
Create embedding features. Recompute PR-AUC and recall@FPR; compare to the baseline.
 
#phase 3 — ablation + quick readout
baseline (tabular) vs +graph embeddings vs (+pair-recency if you kept it separate).
show: PR-AUC, recall@0.5% FPR, and a calibration curve or top-k precision.
feature importances (gain) to sanity-check signal (pair_count, sec_since_last, degree features should rank high).

#phase 4 — if time remains
simple contrastive embedding: treat (sender window of 5 tx) as “anchor”, positive = later window of same sender, negatives = other senders; train a tiny 2-layer MLP on aggregated stats to separate them. Use the learned hidden as a user embedding and plug into the model.
why this works for “semantics”
temporal/velocity + pair familiarity exposes sequential dependencies (who pays whom, how often, and when).
random-walk embeddings capture neighborhoods in the payments graph (shared counterparts → similar vectors).
together they approximate the “context” idea from the tweet without needing billions of tx.