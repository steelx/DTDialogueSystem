# DTDialogueSystem
A Easiest Dialogue system for Unreal Engine which uses Data Tables (@ajinkyax)

HOW TO VIDEO: [https://www.youtube.com/watch?v=66Xw8L9fs6M](https://youtu.be/Efqme7inalk)

# DTDialogueSystem Plugin Documentation

## 1. Overview

Welcome to the DTDialogueSystem! This plugin provides a powerful, flexible, and data-driven dialogue system for Unreal Engine. It's designed to be intuitive for designers using Blueprints while remaining extensible for programmers working in C++.

The core philosophy is to manage complex conversations, state changes, and gameplay events through simple Data Tables. This allows you to create branching dialogues and quests without writing complex code.

### Core Features:
- **Data-Driven:** All dialogue lines, choices, and events are stored in Data Tables.
- **Event-Based:** Use dialogue choices to fire named events, which can be used to trigger anything in your game (e.g., update a quest, open a door, give an item).
- **Stateful Conversations:** NPCs can remember past conversations and change their dialogue based on world state or player actions (we call these "Facts").
- **Blueprint & C++:** Easy to use in Blueprints, with full C++ source for extension.
- **Decoupled Interaction:** The plugin does not handle player interaction (like tracing for NPCs), making it compatible with any interaction system you build.

---

## 2. Core Components & Flow

The system is built around a few key components that you add to your character Blueprints.

| Component | Who gets it? | Purpose |
| :--- | :--- | :--- |
| **DialogueComponent** | Player & NPC | The "brain" of the system. It holds dialogue data and broadcasts/receives dialogue events. |
| **InitiateDialogueComponent** | Player | Provides the `TryStartDialogueWith` function. It is called by your own interaction system to begin a conversation with a specific target actor. |
| **ShowDialoguePromptComponent**| NPC | (Optional) A component to show a tooltip widget (e.g., "Press E to Talk") above the NPC's head. You control when to show/hide it. |

### How a Conversation Starts:

The dialogue plugin is now decoupled from the interaction system. Your game is responsible for finding an interactable actor. Once you have a target, you use the `InitiateDialogueComponent` to start the conversation.

Here is the new, recommended flow:

```ascii
+--------------------------+                   +--------------------------+
|      PLAYER              |                   |      NPC                 |
|--------------------------|                   |--------------------------|
|                          |                   |                          |
|  YourInteractionComponent|---(1. Trace)--->  | (Any Actor with a        |
|  (Finds a target actor)  |                   |  DialogueComponent)      |
|                          |                   |                          |
|      (Player Presses E)  |                   |                          |
|                          |                   |                          |
| PrimaryInteract() called!|                   |                          |
|           |              |                   |                          |
+-----------|--------------+                   +--------------------------+
            |
            v
+--------------------------+
| InitiateDialogueComponent|
|--------------------------|
|                          |
| TryStartDialogueWith()   |---(2. Start)--->  | DialogueComponent        |
|      (called with        |                   | (on the traced NPC)      |
|       traced NPC)        |                   |                          |
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

1.  **Your Code Traces:** Your game's own interaction component (e.g., one you create in C++ or Blueprints) performs a trace to find a "focused" actor.
2.  **Your Code Initiates:** When the player presses the interact key, your code calls the `TryStartDialogueWith` function on the player's `InitiateDialogueComponent`, passing in the actor you found.
3.  **Dialogue Manager Takes Over:** If the target is a valid dialogue participant, the `DialogueManager` starts the conversation and displays the UI.

> **Important Note:** A common question is: If an event fires, how does the system know which NPC to update? The `DialogueManager` tracks the participants of the **active** conversation. When an event is handled, it targets the NPC involved in that specific dialogue session, ensuring other NPCs in the world are not accidentally affected.

---

## 3. SETUP

Here’s how to get your characters ready for dialogue.

### 3.1. NPC Setup

Any character that can talk needs a `DialogueComponent`.

1.  **Open your NPC's Blueprint.**
2.  **Add Component**: Click the `+ Add` button and add a `Dialogue Component`.
3.  **Configure the `Dialogue Component`**:
    *   **Participant Name**: Give the NPC a unique name (e.g., `NPC1` or `Hinter`).
    *   **Conditional Start Points**: This is where you'll define what dialogue the NPC starts with. We'll cover this in the example below.
4.  **(Optional) Add a `ShowDialoguePromptComponent`** if you want to display a widget above the NPC's head. You are responsible for calling `ShowPrompt()` and `HidePrompt()` from your own interaction code when the NPC gains or loses focus.

### 3.2. Player Setup (Recommended Pattern)

The player character needs a `DialogueComponent` (to handle events) and an `InitiateDialogueComponent` (to start conversations). The most flexible way to handle interaction is to use a Blueprint Interface.

#### Step 1: Create an Interact Interface

1.  In the Content Browser, go to `Add > Blueprints > Blueprint Interface`.
2.  Name it `BPI_Interact`.
3.  Open it and create a new function named `Interact`. Give it one input: an `Actor` reference named `Instigator`.

#### Step 2: Add the Interface to your NPC

1.  Open your NPC Blueprint.
2.  Go to `Class Settings` and under the `Interfaces` panel, click `Add` and select your `BPI_Interact` interface.
3.  This will add an "Event Interact" node to your Event Graph.

#### Step 3: Configure the Player Character

1.  **Open your Player's Blueprint.**
2.  **Add Components**: Click `+ Add` and add:
    *   `Dialogue Component`
    *   `Initiate Dialogue Component`
3.  **Build the Interaction Logic**:
    *   Create your own tracing and interaction logic. For example, in your Player Character's `Tick` event, you could do a line trace from the camera to find an actor.
    *   When the player presses the "Interact" key, get the actor you found from your trace.
    *   Call the `Interact` message on that actor, passing in a reference to the Player Character (`Self`).

    *(This part is up to your game's design. The key is that your code finds a target and sends a message).*

#### Step 4: Implement the NPC's `Event Interact`

This is where the magic happens. The NPC decides what "interact" means.

1.  **Open your NPC's Event Graph.**
2.  Find the **`Event Interact`** node.
3.  From the `Instigator` pin (this is the Player), drag out a wire and use **`Get Component by Class`**, selecting `InitiateDialogueComponent`.
4.  From the component reference, call **`Try Start Dialogue With`**.
5.  For the `Target Actor` pin, plug in a reference to **`Self`**.

This setup is extremely powerful. Your player code doesn't know about dialogue, it just "interacts." The NPC decides that interacting with it should start a conversation.

**Example NPC `Event Interact` Graph:**
<img width="1844" height="1002" alt="NPC_BP_start-dialogue" src="https://github.com/user-attachments/assets/68c37cf2-6514-4f59-8950-2447d605916c" />


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

