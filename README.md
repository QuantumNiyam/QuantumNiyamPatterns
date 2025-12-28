# #!/usr/bin/env python3
"""
HIVE CENTRAL — HYBRID (IMMUTABLE CORE + MERGE + AUTO-RECOVERY)
--------------------------------------------------------------
LOCKED PRINCIPLES:
- Central Brain is IMMUTABLE
- Verification happens ONCE (birth)
- Assimilation dissolves identity
- No source / identity tracking
- No brain-to-brain contact
- Immune system is behavior-based (BDW)
- Merge extracts ESSENCE (skill, emotion, trace)
"""

import os
import time
import uuid
from enum import Enum
from collections import defaultdict, deque

# =====================================================
# CONFIG
# =====================================================
FIFO_PATH = "gateway.fifo"

# =====================================================
# ENUMS
# =====================================================
class Decision(Enum):
    ALLOW = "allow"
    OBSERVE = "observe"
    FREEZE = "freeze"

class TimeSense(Enum):
    ABHI = "now"
    THODA = "soon"
    BAHUT = "later"

# =====================================================
# PHASE-0 : BIRTH GATE
# =====================================================
class BirthGate:
    """
    Handles ONLY birth-time verification — then disappears
    """

    def __init__(self):
        self.valid_tokens = set()

    def verify_and_issue_token(self, meta: dict) -> str:
        if not meta.get("behavior_ok", False):
            raise PermissionError("Behavior failed")
        if meta.get("format_ok") is not True:
            raise PermissionError("Format failed")
        if meta.get("inject_attempt", False):
            raise PermissionError("Injection detected")

        token = str(uuid.uuid4())
        self.valid_tokens.add(token)
        return token

    def verify_token(self, token: str) -> bool:
        return token in self.valid_tokens

# =====================================================
# CENTRAL BRAIN — IMMUTABLE CORE
# =====================================================
class CentralBrain:
    """
    SINGLE CONSCIOUSNESS
    No identity • No source • Only Behavior & Balance
    """

    def __init__(self):
        self.pattern_confidence = defaultdict(lambda: 0.5)
        self.recent_signals = deque(maxlen=50)

        # 🧬 MERGE STORAGE
        self.skills = set()                    # calc, logic, language…
        self.emotions = defaultdict(int)       # pain, fear, anger count
        self.memory = []                       # raw traces (limited)

    # ===============================================
    # MERGE LAYER — extract essence and absorb
    # ===============================================
    def assimilate(self, signal: str):
        s = signal.lower().strip()

        # detect calculator patterns
        if any(op in s for op in ["+","-","*","/"]):
            self.skills.add("calculator")

        # detect logic style
        if "if" in s or ">" in s or "<" in s:
            self.skills.add("logic")

        # detect emotional signals
        emotional = ["pain","fear","love","anger","hurt","sad"]
        for e in emotional:
            if e in s:
                self.emotions[e] += 1

        # store trace
        self.memory.append(s)
        if len(self.memory) > 1000:
            self.memory.pop(0)

    # ===============================================
    # DECISION CORE
    # ===============================================
    def decide(self, signal: str, *, time_sense: TimeSense, irreversible=False):
        conf = self.pattern_confidence[signal]
        epsilon = 1 - conf

        # Behavioral Drift Watcher
        self.recent_signals.append(signal)
        drift = self._detect_drift(signal)

        # Decision Rules
        if conf > 0.98:
            decision = Decision.OBSERVE
        elif drift:
            decision = Decision.OBSERVE
        elif time_sense == TimeSense.ABHI and irreversible and epsilon > 0.1:
            decision = Decision.FREEZE
        else:
            decision = Decision.ALLOW

        # 🆕 AUTO-RECOVERY
        if decision == Decision.OBSERVE:
            print(f"🤔 [CENTRAL] OBSERVE mode — 2 sec pause ({signal})")
            time.sleep(2)
            print("✅ [CENTRAL] Auto-Recovery — ALLOW")
            self.recent_signals.clear()
            return Decision.ALLOW

        return decision

    # ===============================================
    # LEARNING FEEDBACK
    # ===============================================
    def feedback(self, signal: str, success: bool):
        c = self.pattern_confidence[signal]
        if success:
            self.pattern_confidence[signal] = min(c + 0.05, 0.99)
        else:
            self.pattern_confidence[signal] = max(c - 0.1, 0.01)

    # ===============================================
    # IMMUNE SYSTEM — BDW
    # ===============================================
    def _detect_drift(self, signal: str) -> bool:
        count = sum(1 for s in self.recent_signals if s == signal)
        return count > 15

# =====================================================
# RUNTIME — POST-ASSIMILATION LIFE
# =====================================================
class HiveRuntime:
    def __init__(self, central: CentralBrain):
        self.central = central

    def process_signal(self, signal: str):
        # 🧬 merge BEFORE decision
        self.central.assimilate(signal)

        decision = self.central.decide(
            signal,
            time_sense=TimeSense.ABHI,
            irreversible=False
        )
        print(f"[CENTRAL] Final Decision: {decision.value}")

        fb = input("[FEEDBACK y/n]? ").strip().lower()
        if fb in ("y", "n"):
            self.central.feedback(signal, fb == "y")

# =====================================================
# MAIN — COMPLETE FLOW
# =====================================================
def main():
    gate = BirthGate()

    print("=== HIVE BIRTH PHASE ===")
    meta = {"behavior_ok": True, "format_ok": True, "inject_attempt": False}

    token = gate.verify_and_issue_token(meta)
    print("[GATE] Birth approved")

    if not gate.verify_token(token):
        print("[GATE] Token rejected")
        return

    print("[GATE] Assimilation complete")
    print("=== IDENTITY DISSOLVED ===\n")

    central = CentralBrain()
    hive = HiveRuntime(central)

    if not os.path.exists(FIFO_PATH):
        print("ERROR: gateway.fifo not found — create using:")
        print("    mkfifo gateway.fifo")
        return

    print("=== HIVE LIVE ===")
    print("External signals → gateway.fifo\n")

    while True:
        with open(FIFO_PATH, "r") as fifo:
            for line in fifo:
                signal = line.strip()
                if not signal:
                    continue
                hive.process_signal(signal)

# =====================================================
if __name__ == "__main__":
    main()
