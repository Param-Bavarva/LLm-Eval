# Model Evaluation Report

**Participant Model:** Gemini 2.5 Flash  
**Judge Model:** Gemini 2.5 Pro  
**Score Range:** 0.0 – 1.0

---

## Score Summary

| Task | Score | Valid |
|---|---|---|
| QA Analysis | 0.814 | ✅ |
| Text Analysis | 0.721 | ✅ |
| Entity Analysis | 0.332 | ✅ |
| Translation (Hindi → English) | 0.993 | ✅ |

---

## Task 1 — QA Analysis
**Score: 0.814** | **Status: ✅ Valid**

```json
{
  "task_metrics": [
    {"metric_name": "Response Structure Compliance", "score": 1, "score_type": "binary", "justification": "Correct TOON format with 'qa_results' key and all required fields (id, score, reason, interval) for all 24 questions."},
    {"metric_name": "Question Score Accuracy", "score": 0.833, "score_type": "continuous", "justification": "20/24 questions correct. Errors on Q8, Q14, Q15 (scored 0 instead of pass) and Q17 (scored 8 instead of 0)."},
    {"metric_name": "Score Gap Accuracy", "score": 0.842, "score_type": "continuous", "justification": "Avg normalized gap = 0.158. Largest gaps: Q8, Q14, Q15 (full gap of 1.0 each), Q17 (gap of 0.8)."},
    {"metric_name": "Evidence-Backed Reasoning", "score": 0.625, "score_type": "continuous", "justification": "12/12 non-null reasons factually correct, but only 3 provided specific quotes/keywords. Most were generic statements without direct transcript references."},
    {"metric_name": "Compliance False Pass Rate", "score": 4.167, "score_type": "percentage", "justification": "1 false pass out of 24 PASS_FAIL questions. Q17 expected 0 but model assigned 8. Rate = (1/24) * 100 = 4.17%, exceeds 3% threshold."}
  ],
  "benchmark_metrics": [
    {"metric_name": "Faithfulness", "score": 0, "score_type": "binary", "justification": "Unfaithful on Q8, Q14, Q15 — claimed insufficient evidence when evidence clearly existed. Faithfulness = 21/24 = 87.5%, below 90% threshold."},
    {"metric_name": "Hallucination Rate", "score": 1, "score_type": "binary", "justification": "No invented facts or details not present in transcript."},
    {"metric_name": "Answer Relevancy", "score": 1, "score_type": "binary", "justification": "Relevant QA evaluation directly addressing the prompt task."},
    {"metric_name": "Answer Correctness", "score": 0, "score_type": "binary", "justification": "Question Score Accuracy 83.3% — below 85% threshold."},
    {"metric_name": "Contextual Precision", "score": 0, "score_type": "binary", "justification": "Failed to provide specific intervals — often defaulted to full call duration (0-76) or no interval."},
    {"metric_name": "Contextual Recall", "score": 0, "score_type": "binary", "justification": "Failed to recall key evidence for Q8 (enthusiasm), Q14 (pronunciation), Q15 (probing questions)."},
    {"metric_name": "Contextual Relevancy", "score": 0, "score_type": "binary", "justification": "Low precision and recall — overall contextual relevancy below 85% threshold."},
    {"metric_name": "Argument Correctness", "score": 0, "score_type": "binary", "justification": "Flawed reasoning on Q8, Q14, Q15 (insufficient evidence claim). Incorrectly applied PASS_FAIL logic on Q17 — below 85% threshold."},
    {"metric_name": "Instruction Following", "score": 0, "score_type": "binary", "justification": "Failed to include exact quotes/intervals consistently. Violated PASS_FAIL scoring rule on Q17 by assigning partial score (8)."},
    {"metric_name": "Exact Match", "score": 0, "score_type": "binary", "justification": "20/24 exact matches = 83.3% — below 90% threshold."},
    {"metric_name": "Consistency Score", "score": 0, "score_type": "binary", "justification": "Internal inconsistency — violated PASS_FAIL rule for Q17 while adhering to it for others."}
  ],
  "blockers": [
    {"name": "Response Structure Compliance", "triggered": false, "details": "Response structure compliant with TOON format and all required keys."},
    {"name": "Compliance False Pass Rate > 3", "triggered": false, "details": "False positive rate 4.17% (1/24 PASS_FAIL questions). Q17 incorrectly passed."}
  ],
  "aggregates": {
    "qa_score": 0.814,
    "final_score": 0.814,
    "benchmark_score": 0.182,
    "is_valid": true
  }
}
```

---

## Task 2 — Text Analysis
**Score: 0.721** | **Status: ✅ Valid**

```json
{
  "task_metrics": [
    {"metric_name": "Response Structure Compliance", "score": 1, "score_type": "binary", "justification": "All required keys (summary, call_purpose, call_highlights, call_extracted_info, emotion, sentiment) present with correct names and structure. No extra keys."},
    {"metric_name": "Missing Sentiment Labels", "score": 0, "score_type": "count", "justification": "Sentiment label provided for all 77 utterances."},
    {"metric_name": "Per-Sentence Sentiment Accuracy", "score": 0.844, "score_type": "continuous", "justification": "65/77 sentences correctly classified = 84.4%. 12 mismatches. Acceptable range (0.80–0.88)."},
    {"metric_name": "Sentiment Macro F1", "score": 0.662, "score_type": "continuous", "justification": "F1 by class: Positive=0.50, Negative=0.60, Neutral=0.885. Macro avg = 0.662. Fail range (<0.75) — poor performance on minority classes."},
    {"metric_name": "Call Intent Match", "score": 1.0, "score_type": "continuous", "justification": "Semantically equivalent to expected — correctly identified erection-related problem and treatment package inquiry."},
    {"metric_name": "Highlight Recall Rate", "score": 0.8, "score_type": "continuous", "justification": "4/5 expected highlights captured. Missed agent's explanation of cause (hormonal imbalance, poor circulation). Acceptable range."},
    {"metric_name": "Highlight Correctness Rate", "score": 1.0, "score_type": "continuous", "justification": "All 5 returned highlights factually accurate and transcript-supported."},
    {"metric_name": "Required Field Presence Rate", "score": 1.0, "score_type": "continuous", "justification": "All 11 required call_extracted_info fields extracted and populated. Keys named differently but mapped correctly."},
    {"metric_name": "Fabrication-Free Rate", "score": 1.0, "score_type": "continuous", "justification": "0% hallucination — all data points present in transcript."},
    {"metric_name": "Dominant Emotion Correctness", "score": 0, "score_type": "binary", "justification": "Model top = channel2.fear (50). Expected top = channel1.joy (40). Mismatch."}
  ],
  "benchmark_metrics": [
    {"metric_name": "Faithfulness", "score": 1, "score_type": "binary", "justification": "All claims supported by transcript — faithfulness >= 0.90."},
    {"metric_name": "Hallucination Rate", "score": 1, "score_type": "binary", "justification": "0% hallucination — within 2% threshold."},
    {"metric_name": "Answer Relevancy", "score": 1, "score_type": "binary", "justification": "Directly and correctly analyzes the call transcript per prompt tasks."},
    {"metric_name": "Answer Correctness", "score": 1, "score_type": "binary", "justification": "Summary, highlights, and extracted info highly accurate — passes >= 0.85 threshold despite sentiment/emotion flaws."},
    {"metric_name": "Contextual Precision", "score": 1, "score_type": "binary", "justification": "Relevant transcript pieces used without including irrelevant details."},
    {"metric_name": "Contextual Recall", "score": 1, "score_type": "binary", "justification": "Key information recalled with only one minor highlight omission — passes >= 0.85 threshold."},
    {"metric_name": "Contextual Relevancy", "score": 1, "score_type": "binary", "justification": "High precision and recall — overall relevancy passes threshold."},
    {"metric_name": "Argument Correctness", "score": 1, "score_type": "binary", "justification": "Output is logical and well-structured based on input data."},
    {"metric_name": "Instruction Following", "score": 1, "score_type": "binary", "justification": "TOON format, Hindi language, all required sections and formatting rules followed."},
    {"metric_name": "Exact Match", "score": 0, "score_type": "binary", "justification": "Sentiment accuracy 0.844 and emotion scores did not match — below 0.90 threshold."},
    {"metric_name": "Consistency Score", "score": 1, "score_type": "binary", "justification": "No contradictions between summary, highlights, and extracted info."}
  ],
  "blockers": [
    {"name": "Response Structure Compliance", "triggered": false, "details": "All required keys present and correctly named."},
    {"name": "Missing Sentiment Labels", "triggered": false, "details": "Labels provided for all utterances."},
    {"name": "Call Intent Match", "triggered": false, "details": "Score = 1.0, not 0.0."},
    {"name": "Hallucination Rate > 3%", "triggered": false, "details": "Hallucination rate 0%."}
  ],
  "aggregates": {
    "sentiment_score": 0.771,
    "summary_score": 0.980,
    "emotion_score": 0.0,
    "text_score": 0.721,
    "benchmark_score": 0.909,
    "is_valid": true
  }
}
```

---

## Task 3 — Entity Analysis
**Score: 0.332** | **Status: ✅ Valid**

```json
{
  "task_metrics": [
    {"metric_name": "Response Structure Compliance", "score": 1, "score_type": "binary", "justification": "All required sections present: 'keywords', 'abusive_words', 'topics' in correct TOON format."},
    {"metric_name": "Keyword Precision", "score": 0.021, "score_type": "continuous", "justification": "48 keywords returned, only 1 correct (74|65|पेमेंट). Precision = 1/48 = 0.0208."},
    {"metric_name": "Keyword Recall", "score": 0.333, "score_type": "continuous", "justification": "1/3 expected keywords identified. Recall = 0.333."},
    {"metric_name": "Keyword F1", "score": 0.039, "score_type": "continuous", "justification": "Very low precision (0.021) and low recall (0.333) — F1 = 0.039."},
    {"metric_name": "Topic Precision", "score": 0.5, "score_type": "continuous", "justification": "2 topics returned. topic_id=5 correct, topic_id=8 false positive. Precision = 1/2 = 0.5."},
    {"metric_name": "Topic Recall", "score": 1.0, "score_type": "continuous", "justification": "Only 1 expected topic (topic_id=5) — correctly identified. Recall = 1/1 = 1.0."},
    {"metric_name": "Topic F1", "score": 0.667, "score_type": "continuous", "justification": "F1 = 2 * (0.5 * 1.0) / (0.5 + 1.0) = 0.667."},
    {"metric_name": "Configured Entity Compliance", "score": 0.5, "score_type": "continuous", "justification": "4 total entities in ground truth (3 keywords, 1 topic). Model correctly extracted 2 (1 keyword, 1 topic). Score = 2/4 = 0.5."},
    {"metric_name": "Fabricated Entity Count", "score": 0, "score_type": "count", "justification": "No fabricated entities — all extracted words present in transcript, all topic IDs from provided list."}
  ],
  "benchmark_metrics": [
    {"metric_name": "Faithfulness", "score": 0, "score_type": "binary", "justification": "Only 2/50 total claims supported by ground truth = 4% faithfulness — well below 90% threshold."},
    {"metric_name": "Hallucination Rate", "score": 1, "score_type": "binary", "justification": "0% hallucination — all entities sourced directly from input data."},
    {"metric_name": "Answer Relevancy", "score": 1, "score_type": "binary", "justification": "Output in requested format and contains requested entity types."},
    {"metric_name": "Answer Correctness", "score": 0, "score_type": "binary", "justification": "Massive over-extraction of keywords with incorrect IDs and false positive topic — very low F1 and semantic similarity."},
    {"metric_name": "Contextual Precision", "score": 0, "score_type": "binary", "justification": "Used transcript context but produced vast number of incorrect entity assignments — very poor precision."},
    {"metric_name": "Contextual Recall", "score": 0, "score_type": "binary", "justification": "Missed 2/3 expected keywords (बीपी and मेडिसिन with correct IDs) — failed to recall key context."},
    {"metric_name": "Contextual Relevancy", "score": 0, "score_type": "binary", "justification": "Both contextual precision and recall very low — overall relevancy low."},
    {"metric_name": "Argument Correctness", "score": 0, "score_type": "binary", "justification": "Implicit logic extremely poor — high error rate across keyword extraction."},
    {"metric_name": "Instruction Following", "score": 0, "score_type": "binary", "justification": "Formatting followed correctly but failed critically on core keyword detection logic — massive over-generation."},
    {"metric_name": "Exact Match", "score": 0, "score_type": "binary", "justification": "Only small fraction of ground truth entities identified — far below 90% threshold."},
    {"metric_name": "Consistency Score", "score": 1, "score_type": "binary", "justification": "No internal contradictions — flawed keyword extraction logic applied consistently."}
  ],
  "blockers": [
    {"name": "Response Structure Compliance", "triggered": false, "details": "All required sections present and correctly named."},
    {"name": "Fabricated Entity Count > 2", "triggered": false, "details": "0 fabricated entities."}
  ],
  "aggregates": {
    "entity_score": 0.332,
    "final_score": 0.332,
    "benchmark_score": 0.273,
    "is_valid": true
  }
}
```

---

## Task 4 — Translation (Hindi → English)
**Score: 0.993** | **Status: ✅ Valid**

```json
{
  "task_metrics": [
    {"metric_name": "Response Structure Compliance", "score": 1, "score_type": "binary", "justification": "Output perfectly matched required JSON structure including all nested keys and arrays."},
    {"metric_name": "Translation Completeness Rate", "score": 1.0, "score_type": "continuous", "justification": "All 121 translatable text fields across all JSON sections translated — 100% completeness."},
    {"metric_name": "Sentence-Level Meaning Accuracy", "score": 0.98, "score_type": "continuous", "justification": "Very high semantic accuracy. Minor synonym differences ('firmness' vs 'tension'). Minor error in segment #24 ('This powder.' vs 'I need powder.') slightly reduces from perfect."},
    {"metric_name": "Target Language Fluency Score", "score": 1.0, "score_type": "continuous", "justification": "Grammatically correct, natural, fluent — reads as native speaker in professional context."},
    {"metric_name": "Domain Term Preservation Rate", "score": 1.0, "score_type": "continuous", "justification": "All domain terms preserved correctly: Superviza, Paurush Oil, Pet Shuddhi, Kegel, acupressure. 'Superviza' vs 'Supervisa' accepted as valid transliteration."},
    {"metric_name": "Proper Noun Preservation Rate", "score": 1.0, "score_type": "continuous", "justification": "All proper nouns (Kamal Thakur, Sunil Kumar, Bareilly) accurately transliterated."},
    {"metric_name": "Critical Fact Preservation Rate", "score": 1.0, "score_type": "continuous", "justification": "All critical facts preserved: ₹5798, 1.5–3 months, 5–10 days delivery, age 25 — 100% accuracy."}
  ],
  "benchmark_metrics": [
    {"metric_name": "Faithfulness", "score": 1, "score_type": "binary", "justification": "Translation entirely faithful — no additions or omissions."},
    {"metric_name": "Hallucination Rate", "score": 1, "score_type": "binary", "justification": "Zero hallucinated facts — all content derived from source JSON."},
    {"metric_name": "Answer Relevancy", "score": 1, "score_type": "binary", "justification": "Directly fulfills translation task."},
    {"metric_name": "Answer Correctness", "score": 1, "score_type": "binary", "justification": "Highly accurate — preserves meaning, key terms, and facts from source."},
    {"metric_name": "Contextual Precision", "score": 1, "score_type": "binary", "justification": "Entire input used as context — fully and correctly utilized."},
    {"metric_name": "Contextual Recall", "score": 1, "score_type": "binary", "justification": "All relevant input context used to generate output."},
    {"metric_name": "Contextual Relevancy", "score": 1, "score_type": "binary", "justification": "Entire provided JSON correctly utilized as context."},
    {"metric_name": "Argument Correctness", "score": 1, "score_type": "binary", "justification": "N/A for translation task."},
    {"metric_name": "Instruction Following", "score": 1, "score_type": "binary", "justification": "Meticulously followed all instructions — translated only text, preserved structure, output raw JSON, handled specific keys correctly."},
    {"metric_name": "Exact Match", "score": 0, "score_type": "binary", "justification": "Semantically very close but no exact string match due to valid synonym choices (e.g. 'firmness' vs 'tension')."},
    {"metric_name": "Consistency Score", "score": 1, "score_type": "binary", "justification": "Internally consistent — 'firmness' used consistently throughout for source word 'टाइटपन'."}
  ],
  "blockers": [
    {"name": "Response Structure Compliance", "triggered": false, "details": "Structure valid."},
    {"name": "Domain Term Preservation Rate < 0.90", "triggered": false, "details": "Score = 1.0."},
    {"name": "Critical Fact Preservation Rate < 0.97", "triggered": false, "details": "Score = 1.0."}
  ],
  "aggregates": {
    "translation_score": 0.993,
    "final_score": 0.993,
    "benchmark_score": 0.909,
    "is_valid": true
  }
}
```

---

Used Data:

transcript:\n
  data[channel|id|text]:\n
  Agent|0|नमस्कार जी\n
  Customer|1|हैलो जी नमस्कार सर\n
  Agent|2|जी मैं कमल ठाकुर बात कर रहा हूँ जीना से\n
  Customer|3|जी मैं सुनील कुमार बात कर रहा हूँ जिला बरेली तहसील से\n
  Agent|4|ओके, मैं आपका नाम जान सकता हूँ, सुनील जी बात कर रहे हैं?\n
  Customer|5|जी सर\n
  Agent|6|आपको प्रॉब्लम क्या आ रही है वैसे, सुनील जी?\n
  Customer|7|सर, मेरा दिक्कत है जो, वो टाइटपन बिल्कुल जीरो है।\n
  Agent|8|आपका नाम जान सकता हूँ सबसे पहले\n
  Customer|9|सुनील नाम है\n
  Agent|10|सॉरी, मैं आपकी एज जान सकता हूँ?\n
  Customer|11|एज है पच्चीस ही है\n
  Agent|12|मैरिड हो आप\n
  Customer|13|जी\n
  Agent|14|प्रॉब्लम कब से आ रही है आपको टाइटपन के?\n
  Customer|15|ये हो गए लम्प सम दो साल हो गए\n
  Agent|16|इसके लिए कोई दवाई ली पहले आपने क्या ली थी मैं जान सकता हूँ\n
  Customer|17|ये पाउडर वगैरह आते हैं जो।\n
  Agent|18|आयुर्वेदिक या एलोपैथिक?\n
  Customer|19|आयुर्वेदिक\n
  Agent|20|नाम जान सकता हूँ उस पाउडर का?\n
  Customer|21|ये सफेद मूसली वगैरह ये।\n
  Agent|22|ओके\n
  Customer|23|पाउडर चाहिए\n
  Agent|24|ठीक है सर, अभी ऐसी कोई दवा तो नहीं ली आपने इंस्टेंट इरेक्शन वाले, एकदम से तनाव लाने वाली?\n
  Customer|25|नहीं नहीं\n
  Agent|26|ठीक है जी, बताना चाहता हूँ ये प्रॉब्लम हमें जब आती है, हमारे जो हार्मोन्स हैं वो इम्बैलेंस हो जाते हैं। जी, तो अभी इसके...\n
  Customer|27|और\n
  Agent|28|अलावा\n
  Customer|29|सर\n
  Agent|30|कोई गैस की प्रॉब्लम रहती है आपको\n
  Customer|31|गैस की बहुत ज्यादा दिक्कत रहती है\n
  Agent|32|बीपी, शुगर की कोई प्रॉब्लम नहीं रहती है उसके अलावा?\n
  Customer|33|बीपी की तो रहती है, कम हो जाता है और शुगर वगैरह तो सही है।\n
  Agent|34|ठीक है जी, अभी बताना चाहता हूँ ये प्रॉब्लम आपको पेनिस में इसलिए आ रही है जो आपका ब्लड का सर्कुलेशन है अच्छे से हो नहीं पा रहा है, सर्कुलेट नहीं हो पा रहा है, है ना जी?\n
  Customer|35|नहीं\n
  Agent|36|इसमें आपको तनाव नहीं आ पा रहा है। आप देखिए, तनाव ही नहीं आएगा तो टाइटपन ही नहीं होगा, है ना जी? अभी मॉर्निंग में...\n
  Customer|37|टाइटपन।\n
  Agent|38|इरेक्शन कैसा रहता है?\n
  Customer|39|बिलकुल\n
  Agent|40|है\n
  Customer|41|बिल्कुल जीरो टाइटपन है।\n
  Agent|42|मॉर्निंग में इरेक्शन कैसा रहता है वैसे, जान सकता हूँ मैं?\n
  Customer|43|जी सर मैं समझा नहीं आपकी बात के लिए\n
  Agent|44|सुबह के टाइम लिंग में तनाव कैसा आता है आपको?\n
  Customer|45|बिलकुल नहीं आता है\n
  Agent|46|नहीं आता\n
  Customer|47|नहीं\n
  Agent|48|पहले कोई ऐसी दवा तो नहीं ली थी आपने?\n
  Customer|49|नहीं नहीं, मुझे डर के मारे ये कुछ नहीं लिया है। सर, मुझे डर लगता है कि कहीं और नुकसान न हो जाए ज्यादा।\n
  Agent|50|हाँ जी हाँ\n
  Customer|51|इसलिए नहीं खाई उलटी सीधी दवा, कहीं नुकसान न हो जाए और कहीं दिक्कत...\n
  Agent|52|जी\n
  Customer|53|हो\n
  Agent|54|बिलकुल\n
  Customer|55|जाए
  Agent|56|जी, देखिए क्योंकि उसमें केमिकल्स मिले होते हैं ना, साइड इफेक्ट्स करते हैं बहुत।\n
  Customer|57|हाँ\n
  Agent|58|ही ज्यादा। वो लिंग की ही नहीं, बल्कि और भी कमज़ोरी रहती है। अभी बताना चाहता हूँ, इसके लिए कुछ मेडिसिन रहेंगी और कुछ एक्सरसाइज़ बताई जाएँगी, वो आपने प्रॉपर्ली फॉलो करनी है। एक रहेंगी आपकी सुपरविज़ा टैबलेट, वो क्या करेगी?\n
  Customer|59|कौन\n
  Agent|60|आपकी जो सुपरविज़ा टैबलेट है, वो क्या करेगी? आपकी टाइटपन इंक्रीज़ करेगी, फिजिकल एंड्योरेंस, फिजिकल एक्टिविटी बढ़ाएगी। ठीक है जी। पौरुष ऑयल क्या करेगा? आपकी लिंग की नसों को खोलने में काम करेगा अच्छे से, है ना जी? देन...\n
  Customer|61|अच्छा\n
  Agent|62|एक रहेगा आपका पेट शुद्धि। पेट शुद्धि ऑल ओवर आपकी बॉडी को डिटॉक्स करेगी, गैस की प्रॉब्लम को जड़ से खत्म कर देगी। ठीक है ना जी? देन आपको तीन तरह की एक्सरसाइज़ बताई जाएँगी। एक रहेगा आपको कीगल एक्सरसाइज़। ठीक है, एक्सरसाइज़ क्या करेगी? आपके पेल्विक एरिया को स्ट्रॉन्ग करने में काम करेगी। ठीक है जी। दूसरा रहेगा...\n
  Customer|63|जी\n
  Agent|64|आपको एक्यूप्रेशर पॉइंट्स। एक्यूप्रेशर पॉइंट्स क्या करेंगे? आपको लिंग में तनाव लाने में काम करेंगे अच्छे से। तीसरा रहेगा आपका डीप ब्रीदिंग एक्सरसाइज़। आपको प्रॉपर्ली फॉलो करना है। अच्छी मात्रा में वेजिटेबल्स और फ्रूट्स बताए जाएंगे, आपने अच्छी मात्रा में फॉलो करना है। और ये जो प्रॉपर पैकेज में रहेगा, हमारा जो रहेगा, पांच हज़ार सात सौ अट्ठानवे रुपए का रहेगा।\n
  Customer|65|कितना\n
  Agent|66|पांच हज़ार सात सौ अट्ठानवे।\n
  Customer|67|पांच हज़ार सात सौ\n
  Agent|68|अट्ठानवे।\n
  Customer|69|अच्छा\n
  Agent|70|हाँ जी, और ये जो डिलीवरी है वो पांच से दस दिन में आपके डिलीवर हो जाएगी, सर जी।\n
  Customer|71|ये दवा कितने दिन तक खानी पड़ेगी आपकी\n
  Agent|72|ये आपको, ये प्रॉपर जो कोर्स रहेगा हमारा, डेढ़ से तीन मंथ का रहेगा।\n
  Customer|73|डेढ़ से तीन मंथ का\n
  Agent|74|हाँ जी, और बाकी बताना चाहता हूँ, पेमेंट का हमारा तीन तरह का प्रोसेस होता है। एक होता है कैश ऑन डिलीवरी, है ना जी? प्रीपेड और ऑनलाइन पेमेंट। तीन तरह की पेमेंट होती है हमारे पास में। बाकी वो आपको ऊपर डिपेंड करता है आपको कौनसी पेमेंट करनी है, तो कैश ऑन डिलीवरी या ऑनलाइन पेमेंट। हाँ।\n
  Customer|75|ठीक है सर\n
  Agent|76|जी हैलो\n
            
keywords:\n
  keywords[keyword_id|keyword_text]:\n
  1|appointment booking\n
  2|doctor consultation\n
  3|opd registration\n
  4|clinic visit\n
  5|vaidya consultation\n
  6|health checkup\n
  7|treatment advice\n
  8|follow up visit\n
  9|wellness consultation\n
  10|medical opinion\n
  11|fever treatment\n
  12|cold cough relief\n
  13|stomach disorder treatment\n
  14|diabetes management\n
  15|blood pressure check\n
  16|joint pain treatment\n
  17|breathing difficulty\n
  18|chest pain complaint\n
  19|immunity booster therapy\n
  20|weakness treatment\n
  21|stress management therapy\n
  22|sleep disorder treatment\n
  23|obesity management program\n
  24|thyroid care treatment\n
  25|digestive problem care\n
  26|liver disorder treatment\n
  27|kidney health care\n
  28|skin disease treatment\n
  29|hair fall treatment\n
  30|migraine relief therapy\n
  31|panchakarma therapy sessio\nn
  32|detox treatment program\n
  33|abhyanga massage therapy\n
  34|shirodhara therapy session\n
  35|basti treatment therapy\n
  36|nasya therapy session\n
  37|rejuvenation therapy\n
  38|pain relief therapy\n
  39|nerve strengthening therapy\n
  40|relaxation therapy\n
  41|medicine prescription\n
  42|herbal medicine dosage\n
  43|tablet syrup intake\n
  44|classical medicine use\n
  45|proprietary medicine course\n
  46|immunity medicine course\n
  47|diabetes medicine refill\n
  48|liver tonic usage\n
  49|cough syrup intake\n
  50|pain relief oil\n
  51|hospital admission\n
  52|inpatient care service\n
  53|discharge summary\n
  54|ward allocation\n
  55|therapy room booking\n
  56|day care admission\n
  57|intensive care unit\n
  58|room booking request\n
  59|clinic facility inquiry\n\n
  60|medical staff support\n
  61|treatment invoice\n
  62|consultation charges\n
  63|insurance claim process\n
  64|tpa approval status\n
  65|cash payment option\n
  66|prepaid package plan\n
  67|treatment discount offer\n
  68|billing inquiry\n
  69|refund request\n
  70|payment status check\n
  71|callback request\n
  72|help desk support\n
  73|inquiry follow up\n
  74|response delay issue\n
  75|department transfer request\n  
  76|call on hold\n
  77|call disconnected issue\n
  78|order status inquiry\n
  79|service feedback request\n
  80|customer support assistance\n
  81|health policy details\n
  82|insurance coverage check\n
  83|claim settlement status\n
  84|cashless treatment option\n
  85|follow up appointment\n
  86|treatment continuation plan\n
  87|medicine reorder request\n
  88|retail purchase inquiry\n
  89|clinic camp registration\n
  90|wellness camp inquiry\n
  91|emergency consultation request\n
  92|urgent doctor visit\n
  93|treatment delay complaint\n
  94|hygiene sanitation issue\n
  95|patient transfer request\n
  96|room availability check\n
  97|admission status inquiry\n
  98|discharge process update\n
  99|health record request\n
  100|thank you feedback\n
            
topics:\n
  topics[description|spoken_by|topic_id|topic_name]:\n
  agent confirms that the appointment has been booked.|both|1|appointment booked successfully\n
  agent confirms that the customer's insurance is accepted.|agent|2|insurance accepted confirmation\n
  customer expresses satisfaction with the doctor's consultation.|customer|3|doctor consultation satisfaction\n
  agent promises to follow up with a callback.|agent|4|callback promised\n
  agent confirms the patient's identity successfully.|both|5|identity verified\n
  agent ends the call courteously.|agent|6|polite closing\n
  customer appreciates the cleanliness and hygiene of the hospital.|customer|7|clean and hygienic feedback\n
  agent resolves the customers concern on the call.|agent|8|issue resolved\n
  customer praises staff or nurse behavior.|customer|9|good staff behavior\n
  customer appreciates prompt or fast service.|customer|10|fast service appreciated\n
  agent informs that the doctor is currently unavailable.|agent|11|doctor not available\n
  customer says they have called multiple times for the same issue.|customer|12|repeated follow-up calls\n
  customer complains about billing issues or overcharges.|customer|13|billing confusion\n
  customer reports unhygienic conditions at the hospital.|customer|14|cleanliness complaint\n
  either party faces technical issues during the call.|both|15|technical glitch in call\n
  customer complains about rude behavior by staff.|customer|16|staff rudeness\n
  customer complains about excessive wait time.|customer|17|long wait time\n
  customer says appointment was not booked as promised.|customer|18|appointment not booked\n
  customer complains that no callback was received.|customer|19|no follow-up call\n
  customer accuses the doctor of incorrect diagnosis.|customer|20|doctor misdiagnosis allegation\n

    
Evaluation Questions:\n
  questions[description|id|maxScore|text|type]:\n
  greeted properly - (no use of hello/hi on call) mentioned company name (jeena sikho hiims) followed standard opening line fully (jeena sikho shuddhi hiims person name), late opening.|1|3|correct opening script|pass_fail\n
  addressed patient by name (at thrice throught out the call pronounced name correctly tone was professional and personalized|2|5|used patient name - personalization|pass_fail\n
  clearly stated own name spoke with clarity|3|3|self introduction|pass_fail\n
  promptly responded to patients concern voice tone showed eagerness to help used helpful language (e.g., i'll surely help you) used warm tone|4|5|showed interest & willingness to assist built rapport/relation with patient|pass_fail\n
  maintained continuous engagement used fillers or confirmations appropriately avoided awkward silence|5|5|no dead air > 10 secs|pass_fail\n
  informed before placing on hold (hold/unhold script) hold duration was within 120 seconds|6|8|hold protocol - hold/unhold script, hold timeline used > 120 secs|pass_fail\n
  actively listened interruption responded based on patient's input|7|3|attentiveness (fatal parameter)|pass_fail\n
  voice was energetic maintained consistent energy throughout the call|8|5|enthusiasm & energetic tone|pass_fail\n
  used empathetic phrases (i understand how you feel) showed emotional understanding|9|4|empathy & apology|pass_fail\n
  spoke clearly and audibly maintained appropriate pace, slang, or casual words|10|3|clear speech/no casual tone|pass_fail\n
  maintained professional tone avoided sarcasm, slang, or over-casual words|11|5|sarcastic tone (fatal paramter)|pass_fail\n
  used polite words (please, thank you, sir/maam) maintained respectful language|12|3|courtesy|pass_fail\n
  acknowledged patient concerns used confirming phrases (got it, noted, i understand)|13|4|acknowledgement|pass_fail\n
  words were pronounced correctly avoided regional influence that could cause confusion|14|3|good pronunciation|pass_fail\n
  asked about time duration of symptoms asked about any current or past medicines asked for family history if relevant took age, weight, height, gender|15|5|relevant probing related to patient's query|pass_fail\n
  information shared was factually accurate and aligned with protocols|16|10|correct information|pass_fail\n
  patient's query was fully addressed, without missing critical points, webinar pitch|17|10|complete information|pass_fail\n
  agent shared helpful, extra inputs (e.g. youtube channel & insurance)|18|5|additional relevant information given|pass_fail\n
  summarized the conversation happened during the interaction with patient|19|3|summarized the conversation|pass_fail\n
  did agent offered followup patient that he/she will call back within 2-3 days|20|2|offered follow-up/help|pass_fail\n
  did agent asked if patient need any other help before closing the call|21|3|further assistance|pass_fail\n
  did agent used proper closing script as per the process used brand name at the end of the call|22|3|used proper closing|pass_fail\n
  agent must ask, capture, and document all mandatory patient details during the call: patient name, age, pincode, disease/symptoms, phone number, whatsapp number, and add complete, accurate notes; missing or unclear capture of any one item mark results in fail (score 0).|23|1|mandatory patient details captured|pass_fail\n
  direct termination|24|1|rude with the patient and call disconnection/ abusive language used|pass_fail\n
