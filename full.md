```mermaid
 sequenceDiagram
    Telephony/App->>+Inbound Service: Audio Stream/Text (Their API)
    Inbound Service->>+Caller History: UserId
    Caller History-->>-Inbound Service: History (PII Redacted)
    Inbound Service->>+Recipe Selector: xMsg(Voice/Text)
    Recipe Selector->>+Recipe Registry: Channel
    Recipe Registry-->>-Recipe Selector: Recipe
    Recipe Selector->>+Broker: Recipe + xMsg
    Broker->>+Recipe Runtime: Recipe + xMsg
    Recipe Runtime->>+Interpretation: xMsg
    Interpretation-->>-Recipe Runtime: xMsg

    critical Interpretation Clear
        note right of Recipe Runtime: Small Model Intent Classifier
        Recipe Runtime->>+Intent Classification: xMsg
        Intent Classification-->>-Recipe Runtime: xMsg(+User Intent)
        Recipe Runtime->>+Recipe Runtime: Verify Intent

        note right of Recipe Runtime: LLM Intent Classifier 
        Recipe Runtime->>+LLM Service: if Intent Unclear
        LLM Service-->>-Recipe Runtime: User Intent
        Recipe Runtime->>+Recipe Runtime: Verify Intent

        option
            note right of Recipe Runtime: RAG Assisted Intent Classification
            Recipe Runtime->>+RAG: xMsg
            
            RAG->>+Document Service: Query
            Document Service->>+Query Embedder: Query
            Query Embedder-->>-Document Service: Q*
            Document Service->>VectorDB: Q*
            Document Service->>VectorDB: Docs
            
            RAG->>+Dataset Service: Query
            Dataset Service->>+VectorDB: Q'*
            VectorDB->>+Dataset Service: Datasets
            Dataset Service-->>-RAG: Datasets
            
            RAG->>+LLM Service: Docs* + Dataset* + Query
            LLM Service->>+RAG: Response Stream

            RAG->>+Recipe Runtime: xMsg(+Next Ask)
            Recipe Runtime->>+Output Generation: xMsg(+Next Ask)
            Output Generation->>+Recipe Runtime: xMsg(Voice/Text)
            Recipe Runtime->>+Broker: xMsg(Voice/Text)
            Broker->>+Outbound Service: xMsg(Voice/Text)
            Outbound Service->>+Telephony/App: Voice/Text

        critical Intent Clear
            note right of Recipe Runtime: Follow up Q and rephrased follow up Q asked for Query Sufficiency
            Recipe Runtime->>+Query Sufficiency: xMsg
            Query Sufficiency->>+Graph Service: Query
            Graph Service->>+Graph DB: Q*
            Graph DB-->>-Graph Service: R*
            Graph Service-->>-Query Sufficiency: Graph
            Query Sufficiency->>+Query Sufficiency: Eval Graph
            Query Sufficiency-->>-Recipe Runtime: xMsg(+Next Ask from User)
            Recipe Runtime->>+Broker: xMsg(Voice/Text)
            Broker->>+Outbound Service: xMsg(Voice/Text)
            Outbound Service->>+Telephony/App: Voice/Text
            
            option
                note right of Recipe Runtime: Query Sufficiency Incomplete despite followups
                    Recipe Runtime->>+LLM Service: Query + History
                    LLM Service-->>+Recipe Runtime: xMsg(+Next Ask)
                    Recipe Runtime->>+Output Generation: xMsg(+Next Ask)
                    Output Generation->>+Recipe Runtime: xMsg(Voice/Text)
                    Recipe Runtime->>+Broker: xMsg(Voice/Text)
                    Broker->>+Outbound Service: xMsg(Voice/Text)
                    Outbound Service->>+Telephony/App: Voice/Text

            option
                note right of Recipe Runtime: Query Sufficiency Done
                    Recipe Runtime->>+Guided Flow Inference: xMsg (if Previous Ask was fulfilled)
                    Guided Flow Inference->>+Recipe Runtime: xMsg(+Next Ask)
                    Recipe Runtime->>+Output Generation: xMsg(+Next Ask)
                    Output Generation->>+Recipe Runtime: xMsg(Voice/Text)
                    Recipe Runtime->>+Broker: xMsg(Voice/Text)
                    Broker->>+Outbound Service: xMsg(Voice/Text)
                    Outbound Service->>+Telephony/App: Voice/Text
            
                    end
    end

```
