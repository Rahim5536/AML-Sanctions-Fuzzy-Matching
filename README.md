# AML Sanctions Fuzzy Matching Engine 🛡️

An automated Python system developed to simulate real-time sanctions screening, Politically Exposed Persons (PEP) matching, and high-risk customer due diligence screening routines.

### ⚙️ Technical Blueprint
- **Data Purification:** Sanitizes unformatted raw text inputs, stripping special characters, titles, and handling variations in white spacing.
- **Fuzzy Name Matching:** Employs string metrics approximating normalized Levenshtein Edit Distance thresholds to trap intentional spelling manipulations or typos.
- **Phonetic Analysis:** Embeds an isolated Soundex algorithm to register matching scores against alternative regional transliterations (e.g., matching "Saeed" to "Said").

```python
"""
====================================================================
PROJECT: AUTOMATED AML SANCTIONS & PEP SCREENING ENGINE
FOCUS:   Fuzzy Name Matching, Phonetics (Soundex), & Risk Scoring
AUTHOR:  Rahim Ahmed
====================================================================
"""

import re
from difflib import SequenceMatcher

# 1. SIMULATED WATCHLIST DATA (OFAC / EU / PEP Mock List)
MOCK_SANCTIONS_LIST = [
    {"id": "W001", "name": "Vladimir Petrov", "type": "Sanctioned Entity", "country": "Russia"},
    {"id": "W002", "name": "Saeed Al-Mansoor", "type": "SDN List", "country": "Iran"},
    {"id": "W003", "name": "John Edward Daw", "type": "PEP / High Risk", "country": "United Kingdom"},
    {"id": "W004", "name": "Maria Rodriguez Gomez", "type": "Narcotics Trafficking", "country": "Colombia"}
]

# 2. DATA PURIFICATION & TEXT CLEANING ACCORDING TO COMPLIANCE STANDARDS
def clean_name(name: str) -> str:
    """Removes special characters, extra spaces, and normalizes name inputs."""
    name = name.upper().strip()
    name = re.sub(r"[^\w\s]", "", name) # Remove punctuation strings
    return " ".join(name.split())

# 3. PHONETIC ALGORITHM (AMERICAN SOUNDEX SUBSET)
def get_soundex(name: str) -> str:
    """Generates Soundex codes to identify phonetic/sound-alike names."""
    name = clean_name(name)
    if not name:
        return "0000"
    
    mapping = {"BFPV": "1", "CGJKQSXZ": "2", "DT": "3", "L": "4", "MN": "5", "R": "6"}
    first_letter = name[0]
    
    code = ""
    for char in name[1:]:
        for key, val in mapping.items():
            if char in key:
                if code and code[-1] == val: # Skip consecutive duplicate codes
                    continue
                code += val
                
    # Filter vowels/letters and pad out/truncate to standard 4 characters
    code = "".join([c for c in code if c not in ["A", "E", "I", "O", "U", "H", "W", "Y"]])
    soundex_code = (first_letter + code + "0000")[:4]
    return soundex_code

# 4. FUZZY MATCH RATIO CALCULATOR (LEVENSHTEIN EDIT DISTANCE APPROXIMATION)
def calculate_fuzzy_ratio(str1: str, str2: str) -> float:
    """Calculates Levenshtein similarity percentage between two strings."""
    return round(SequenceMatcher(None, str1, str2).ratio() * 100, 2)

# 5. ENTERPRISE SCREENING ENGINE LOGIC
def screen_customer(customer_name: str, threshold: float = 75.0):
    """Screens input names against watchlist database via fuzzy and phonetic logic arrays."""
    cleaned_customer = clean_name(customer_name)
    customer_soundex = get_soundex(cleaned_customer)
    
    print(f"\n🔍 RUNNING SEARCH FOR: '{customer_name}'")
    print(f"🧬 Normalized Vector: {cleaned_customer} | Soundex Hash: {customer_soundex}")
    print("-" * 75)
    
    alerts_triggered = 0
    
    for entity in MOCK_SANCTIONS_LIST:
        cleaned_watchlist_name = clean_name(entity["name"])
        watchlist_soundex = get_soundex(cleaned_watchlist_name)
        
        # Calculate matching scores
        fuzzy_score = calculate_fuzzy_ratio(cleaned_customer, cleaned_watchlist_name)
        phonetic_match = (customer_soundex == watchlist_soundex)
        
        # Risk Evaluation Model
        if fuzzy_score >= threshold or phonetic_match:
            alerts_triggered += 1
            risk_level = "🚨 HIGH RISK" if fuzzy_score >= 90 else "⚠️ MEDIUM RISK"
            
            print(f"[{risk_level} ALERT TRIGGERED]")
            print(f" -> Matched Target: {entity['name']} ({entity['type']} - {entity['country']})")
            print(f" -> Fuzzy Match Score: {fuzzy_score}%")
            print(f" -> Phonetic Sound-Alike Match: {phonetic_match}\n")
            
    if alerts_triggered == 0:
        print("✅ CLEAR: No matching targets found above compliance threshold boundaries.\n")

# 6. SIMULATING COMPLIANCE RUNS (TESTING MISSSPELLINGS AND ALIASES)
if __name__ == "__main__":
    # Test Scenario A: Typo entry matching 'Vladimir Petrov'
    screen_customer("Wladimir Petrove")
    
    # Test Scenario B: Phonetic variation matching 'Saeed Al-Mansoor'
    screen_customer("Said Al Mansoor")
    
    # Test Scenario C: Clean individual record with no matching risks
    screen_customer("Rahim Ahmed")
```
