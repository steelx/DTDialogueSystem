# DTDialogueSystem
A Easiest Dialogue system for Unreal Engine which uses Data Tables (@ajinkyax)

HOW TO VIDEO: https://www.youtube.com/watch?v=66Xw8L9fs6M

# DTDialogueSystem Plugin Documentation

## 1. Overview

Welcome to the DTDialogueSystem! This plugin provides a powerful, flexible, and data-driven dialogue system for Unreal Engine. It's designed to be intuitive for designers using Blueprints while remaining extensible for programmers working in C++.

The core philosophy is to manage complex conversations, state changes, and gameplay events through simple Data Tables. This allows you to create branching dialogues and quests without writing complex code.

### Core Features:
- **Data-Driven:** All dialogue lines, choices, and events are stored in Data Tables.
- **Event-Based:** Use dialogue choices to fire named events, which can be used to trigger anything in your game (e.g., update a quest, open a door, give an item).
- **Stateful Conversations:** NPCs can remember past conversations and change their dialogue based on world state or player actions (we call these "Facts").
- **Blueprint & C++:** Easy to use in Blueprints, with full C++ source for extension.
- **Built-in Highlighting:** A simple component to make your NPCs glow or show an outline when they are interactable.

---

## 2. Core Components & Flow

The system is built around a few key components that you add to your character Blueprints.

| Component | Who gets it? | Purpose |
| :--- | :--- | :--- |
| **DialogueComponent** | Player & NPC | The "brain" of the system. It holds dialogue data and broadcasts/receives dialogue events. |
| **InitiateDialogueComponent** | Player | Traces for nearby NPCs and allows the player to start a conversation by calling the `Interact()` function. |
| **HighlightableComponent** | NPC | (Optional & If you add `ShowDialoguePromptComponent` it gets added automatically) Makes the NPC's mesh glow or show an outline when the player is nearby. |
| **ShowDialoguePromptComponent**| NPC | (Optional) Works with the `HighlightableComponent` to show a tooltip widget (e.g., "Press E to Talk") above the NPC's head. |

### How a Conversation Starts:

Here is the typical flow of an interaction:

```ascii
+--------------------------+                   +--------------------------+
|      PLAYER              |                   |      NPC                 |
|--------------------------|                   |--------------------------|
|                          |                   |                          |
| InitiateDialogueComponent|---(1. Trace)--->  | HighlightableComponent   |
|                          |                   | (Shows highlight/prompt) |
|                          |                   |                          |
|      (Player Presses E)  |                   |                          |
|                          |                   |                          |
| Interact() is called!    |---(2. Interact)---> | DialogueComponent        |
|                          |                   | (Starts Dialogue)        |
+--------------------------+                   +--------------------------+
         |                                                ^
         |                                                |
         +-----------------(3. Dialogue Starts)-----------+
                                     |
                                     v
                         +-----------------------+
                         |    DialogueManager    |
                         |  (Game Subsystem)     |
                         +-----------------------+
```

1.  The **Player's** `InitiateDialogueComponent` constantly looks for actors with a `DialogueComponent`. When it finds one, it tells the **NPC's** `HighlightableComponent` to turn on.
2.  The Player presses the "Interact" key (e.g., 'E'), which calls the `Interact()` function on their `InitiateDialogueComponent`.
3.  This starts the dialogue with the focused NPC, using the global `DialogueManager` to display the UI and handle the conversation flow.

> **Important Note:** A common question is: If an event fires, how does the system know which NPC to update? The `DialogueManager` tracks the participants of the **active** conversation. When an event is handled, it targets the NPC involved in that specific dialogue session, ensuring other NPCs in the world are not accidentally affected.

---

## 3. SETUP

Here’s how to get your characters ready for dialogue.

### 3.1. NPC Setup

Any character that can talk needs a few components.

1.  **Open your NPC's Blueprint.**
2.  **Add Components**: Click the `+ Add` button in the Components panel and add the following:
    *   `Dialogue Component`
    *   `Highlightable Component`
    *   `Show Dialogue Prompt Component`

3.  **Configure the Components**:
    *   **Select the `Dialogue Component`**:
        *   **Participant Name**: Give the NPC a unique name (e.g., `NPC1` or `Hinter`).
        *   **Conditional Start Points**: This is where you'll define what dialogue the NPC starts with. We'll cover this in the example below.
    *   **Select the `Highlightable Component`**:
        *   **Outline Material / Glow Material**: Assign the `M_Outline` and `M_Glow` materials provided with the plugin. This controls how the NPC looks when highlighted.
    *   **Select the `Show Dialogue Prompt Component`**:
        *   **Tooltip Widget Class**: Assign a widget to show above the NPC's head, like the provided `WBP_MyTooltipWidget`.

### 3.2. Player Setup

The player character needs two components to initiate conversations.

1.  **Open your Player's Blueprint.**
2.  **Add Components**: Click `+ Add` and add:
    *   `Dialogue Component` (This is so the player can receive events, like "QuestUpdated").
    *   `Initiate Dialogue Component`
> Initial Dialogue Component;s : Dialogue Trace Channel : <Must be set to what NPC is using>
<img width="1742" height="1051" alt="{F43BB428-F6CF-4C83-8F54-E9FE7E8010CC}" src="https://github.com/user-attachments/assets/aaa12ec6-ecbc-4f09-bc97-8a0bdf3d4dd7" />

3.  **Configure Input**:
    *   Go to the **Event Graph** of your Player Blueprint.
    *   Right-click and add your "Interact" input event (e.g., `EnhancedInputAction IA_Interact` or the standard `Keyboard E`).
    *   Drag a reference to the `Initiate Dialogue Component` onto the graph.
    *   From the component reference, drag a wire and call the `Interact` function.

---

## 4. Tutorial: The Multi-NPC "3 Apples" Quest

This advanced tutorial will walk you through a quest involving two NPCs to demonstrate how dialogue events can create a dynamic, interconnected world. We will also introduce a clear naming convention for your data table rows to keep your project organized.

**The Scenario:**
*   **NPC1 (Quest Giver)** asks the Player to find 3 apples.
*   **NPC2 (Hinter)** knows where the last apple is, but will only give a hint if the quest is active. Otherwise, he'll just say hello.

### 4.1: Naming Convention

To keep data tables clean, we recommend the following prefixes:

| Type | Prefix | Example |
| :--- | :--- | :--- |
| **Dialogue Line** | `D_` | `D_NPC1_StartQuest` |
| **Player Choice** | `C_` | `C_AcceptQuest` |
| **Event** | `E_` | `E_AcceptQuest` |
| **State/Fact** | `S_` | `S_AppleQuestAccepted` |

### 4.2: Create the Data Tables

Create three **Data Tables** as before: `DT_DialogueLines`, `DT_PlayerChoices`, and `DT_StateChanges`.

### 4.3: Populate the Data Tables

#### A. `DT_DialogueLines`

| RowName | SpeakerType | DialogueText | PlayerChoices (Array) |
| :--- | :--- | :--- | :--- |
| **D_NPC1_Start** | NPC | Can you get me 3 apples? I'd really appreciate it. | 0: `C_AcceptQuest`<br>1: `C_DeclineQuest` |
| **D_NPC1_Acknowledge**| NPC | Great! Let me know when you have them. | |
| **D_NPC1_InProgress** | NPC | Still looking for those 3 apples? | 0: `C_TurnInApples` |
| **D_NPC1_Thanks** | NPC | You got them all! Thank you so much! | |
| **D_NPC2_Generic** | NPC | Hello there, nice day for a walk. | |
| **D_NPC2_Hint** | NPC | Looking for something? I saw a shiny red apple fall off the cart over there. | |

#### B. `DT_PlayerChoices`

| RowName | ChoiceText | NextLineID | Conditions (Array) | Events (Array) |
| :--- | :--- | :--- | :--- | :--- |
| **C_AcceptQuest** | "Sure, I'll get them for you." | `D_NPC1_Acknowledge` | | 0: `E_AcceptAppleQuest` |
| **C_DeclineQuest**| "Sorry, I can't right now." | | | |
| **C_TurnInApples** | "Here are the 3 apples." | `D_NPC1_Thanks` | 0: `S_PlayerHasApple1`<br>1: `S_PlayerHasApple2`<br>2: `S_PlayerHasApple3` | 0: `E_CompleteAppleQuest` |

*   **Key Point**: `C_TurnInApples` will ONLY appear if the player has all three "Facts".

#### C. `DT_StateChanges`

This table is the "brain" of our quest logic.

| RowName | FactsToAdd (Array) | FactsToRemove (Array) | NewConditionalDialogue |
| :--- | :--- | :--- | :--- |
| **E_AcceptAppleQuest** | 0: `S_AppleQuestAccepted` | | **RequiredFacts**: [`S_AppleQuestAccepted`]<br>**StartDialogueLineID**: `D_NPC1_InProgress` |
| **E_CompleteAppleQuest** | 0: `S_AppleQuestCompleted` | 0: `S_AppleQuestAccepted`<br>1: `S_PlayerHasApple1`<br>2: `S_PlayerHasApple2`<br>3: `S_PlayerHasApple3` | **RequiredFacts**: [`S_AppleQuestCompleted`]<br>**StartDialogueLineID**: `D_NPC1_Thanks` |

### 4.4: Configure the NPCs

This is where we make the two NPCs aware of the quest state.

#### A. Configure NPC1 (The Quest Giver)

1.  Open the `NPC1` Blueprint and select its `Dialogue Component`.
2.  In **Conditional Start Points**, add one entry. This is his default state.
    *   **Element 0**:
        *   **RequiredFacts**: (empty)
        *   **StartDialogueLineID**: `D_NPC1_Start`
3.  When the player accepts the quest, the `E_AcceptAppleQuest` event will fire, and our `DT_StateChanges` table will automatically add the next dialogue state (`D_NPC1_InProgress`) to this NPC's component at runtime.

#### B. Configure NPC2 (The Hinter)

1.  Open the `NPC2` Blueprint and select its `Dialogue Component`.
2.  In **Conditional Start Points**, we will add **two** entries. The system checks these from top-to-bottom (index 0, then 1, etc.), so order matters!
    *   **Element 0 (Highest Priority)**:
        *   **RequiredFacts**: [`S_AppleQuestAccepted`]
        *   **StartDialogueLineID**: `D_NPC2_Hint`
    *   **Element 1 (Default/Fallback)**:
        *   **RequiredFacts**: (empty)
        *   **StartDialogueLineID**: `D_NPC2_Generic`

> **This is the key to interconnected NPCs!** We are not directly telling NPC2 to change. We are setting a global fact (`S_AppleQuestAccepted`), and NPC2 has been configured to react to it. When the player talks to NPC2, the system will first check if `S_AppleQuestAccepted` is true. If it is, he'll give the hint. If not, it moves to the next entry and uses the generic dialogue.

### 4.5: Create the Apple Blueprint

This is the same as before, just with the new Fact naming convention. Create a `BP_Apple` actor that, when collected, adds a `Fact` to the `DialogueStateSubsystem`.

1.  Create a `BP_Apple` actor.
2.  Add a **Name** variable called `FactToAdd`, and make it **Instance Editable**.
3.  On `BeginOverlap` with the player, get the `DialogueStateSubsystem` and call `Add Fact`, passing in the `FactToAdd` variable. Then destroy the actor.
4.  Place three apples in your level. Set their `FactToAdd` names to `S_PlayerHasApple1`, `S_PlayerHasApple2`, and `S_PlayerHasApple3`.

### 4.6: You're Done!

Now you can test the full, dynamic flow:
1.  Talk to **NPC2**. He will say his generic line (`D_NPC2_Generic`).
2.  Talk to **NPC1**. He will offer the quest (`D_NPC1_Start`).
3.  Accept the quest. The `E_AcceptAppleQuest` event fires, adding the `S_AppleQuestAccepted` fact to the world.
4.  Talk to **NPC2** again. Now that `S_AppleQuestAccepted` is true, he will give the hint about the apple near the cart (`D_NPC2_Hint`).
5.  Collect all three apples.
6.  Return to **NPC1**. He will see the `S_AppleQuestAccepted` fact and use the `D_NPC1_InProgress` dialogue, allowing you to turn in the apples.
7.  Completing the quest fires `E_CompleteAppleQuest`, which cleans up the old facts and gives NPC1 his final "thank you" dialogue state.

