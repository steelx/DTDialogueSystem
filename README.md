# DTDialogueSystem
A Easiest Dialogue system for Unreal Engine which uses Data Tables (@ajinkyax)

HOW TO VIDEO: https://www.youtube.com/watch?v=66Xw8L9fs6M
(Scroll to Section 3 for Setup)

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

---

## 3. SETUP

Here’s how to get your characters ready for dialogue.

### 3.1. NPC Setup

Any character that can talk needs a few components.

1.  **Open your NPC's Blueprint.**
<img width="623" height="326" alt="{CFFB260A-1BF2-40AB-A810-17D1A750864A}" src="https://github.com/user-attachments/assets/7d4ad3e5-1983-45a8-9ee3-dc6fc54a5a94" />

2.  **Add Components**: Click the `+ Add` button in the Components panel and add the following:
    - A   `Dialogue Component`
<img width="770" height="277" alt="{66B5E9D0-6E6B-443E-A4E2-23EC1E86891E}" src="https://github.com/user-attachments/assets/37d4ee69-7c48-4133-8e0a-7f380b704b9c" />

    - C   `Show Dialogue Prompt Component`
<img width="767" height="415" alt="{DAAA4028-8BEC-4190-8A78-3C68B305224B}" src="https://github.com/user-attachments/assets/15bc2f14-4a2d-4f64-9f1d-a730b57df99e" />


3.  **Configure the Components**:
    *   **Select the `Dialogue Component`**:
        *   **Participant Name**: Give the NPC a unique name (e.g., `Shopkeeper`).
        *   **Conditional Start Points**: This is where you'll define what dialogue the NPC starts with. We'll cover this in the example below.
    *   **Select the `Highlightable Component`**:
        *   **Outline Material / Glow Material**: Assign the `M_Outline` and `M_Glow` materials provided with the plugin. This controls how the NPC looks when highlighted.
    *   **Select the `Show Dialogue Prompt Component`**:
        *   **Tooltip Widget Class**: Assign a widget to show above the NPC's head, like the provided `WBP_MyTooltipWidget`.

Your NPC's component setup should look like this:

```ascii
+-----------------------------+
|      Components (NPC)       |
+-----------------------------+
|                             |
|  > CapsuleComponent (Inherited) |
|  > Mesh (Inherited)         |
|      ...                  |
|  + Dialogue Component       |
|  + Highlightable Component  |
|  + Show Dialogue Prompt Cmp |
|                             |
+-----------------------------+
```

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

Your player's input graph should look like this:
<img width="1730" height="793" alt="{72CF3993-D271-4AE2-A921-E7DB3D617564}" src="https://github.com/user-attachments/assets/80268f6a-c59f-4943-982d-f9b7065cab69" />

```ascii
+---------------------------+     +-----------------------------+
|                           |     |                             |
| (Event) IA_Interact       |-----> Get InitiateDialogueComponent|
| [Triggered]-------------->------>                             |
|                           |     +-----------------------------+
+---------------------------+                   |
                                                |
                                                v
                                      +-----------------+
                                      |                 |
                                      |   Interact()    |
                                      |                 |
                                      +-----------------+
```

---

## 4. Tutorial: The "3 Apples" Quest

This example will walk you through creating a simple quest where an NPC asks for 3 apples. This will teach you about **Dialogue Events** and **Facts**.

*   **Events**: Actions that happen when a player makes a choice (e.g., `AcceptQuest`).
*   **Facts**: Pieces of information about the world state (e.g., `PlayerHasApple`).

### 4.1: Create the Data Tables

First, we need to create the Data Tables that will hold our dialogue and logic. In your Content Browser, right-click and create three **Data Tables**.

1.  **`DT_DialogueLines`**:
    *   **Row Struct**: `DialogueData`
    *   This table holds every line of dialogue an NPC can say.

2.  **`DT_PlayerChoices`**:
    *   **Row Struct**: `PlayerChoice`
    *   This table holds every choice a player can make.

3.  **`DT_StateChanges`**:
    *   **Row Struct**: `DialogueStateChange`
    *   This is the "rulebook". It tells the system how to change the game state when an **Event** occurs.

### 4.2: Populate the Data Tables

Now, let's fill in the rows for our apple quest.

#### A. `DT_DialogueLines`

| RowName | SpeakerType | DialogueText | PlayerChoices (Array) |
| :--- | :--- | :--- | :--- |
| **NPC_Start** | NPC | Can you get me 3 apples? I'd really appreciate it. | 0: `Choice_Accept`<br>1: `Choice_Decline` |
| **NPC_Acknowledge**| NPC | Great! Let me know when you have them. | |
| **NPC_InProgress** | NPC | Still looking for those 3 apples? | 0: `Choice_TurnInApples` |
| **NPC_Thanks** | NPC | You got them all! Thank you so much! | |

#### B. `DT_PlayerChoices`

| RowName | ChoiceText | NextLineID | Conditions (Array) | Events (Array) |
| :--- | :--- | :--- | :--- | :--- |
| **Choice_Accept** | "Sure, I'll get them for you." | `NPC_Acknowledge` | | 0: `AcceptAppleQuest` |
| **Choice_Decline**| "Sorry, I can't right now." | | | |
| **Choice_TurnInApples** | "Here are the 3 apples." | `NPC_Thanks` | 0: `PlayerHasApple1`<br>1: `PlayerHasApple2`<br>2: `PlayerHasApple3` | 0: `CompleteAppleQuest` |

*   **Key Point**: `Choice_TurnInApples` will ONLY appear if the player has all three "Facts" (`PlayerHasApple1`, `PlayerHasApple2`, `PlayerHasApple3`). Selecting it fires the `CompleteAppleQuest` event.

#### C. `DT_StateChanges`

This table links the **Events** from `DT_PlayerChoices` to actual changes in the game.

| RowName | FactsToAdd (Array) | FactsToRemove (Array) | NewConditionalDialogue |
| :--- | :--- | :--- | :--- |
| **AcceptAppleQuest** | 0: `PlayerAcceptedAppleQuest` | | **RequiredFacts**: [`PlayerAcceptedAppleQuest`]<br>**StartDialogueLineID**: `NPC_InProgress` |
| **CompleteAppleQuest** | 0: `PlayerFinishedAppleQuest` | 0: `PlayerAcceptedAppleQuest`<br>1: `PlayerHasApple1`<br>2: `PlayerHasApple2`<br>3: `PlayerHasApple3` | **RequiredFacts**: [`PlayerFinishedAppleQuest`]<br>**StartDialogueLineID**: `NPC_Thanks` |

### 4.3: Configure the NPC

1.  Go back to your **NPC Blueprint** and select the `Dialogue Component`.
2.  Find the **Conditional Start Points** property. This array determines which dialogue to start based on the **Facts** the system knows about. The system checks this list from top to bottom and uses the FIRST one that has its conditions met.
3.  Add an entry to the array for the very beginning of the quest. Since it has no `RequiredFacts`, it will be the default starting point.
    *   **Element 0**:
        *   **RequiredFacts**: (empty)
        *   **StartDialogueLineID**: `NPC_Start`

When the player accepts the quest, the `DT_StateChanges` table will automatically add the other conditional start points (`NPC_InProgress` and `NPC_Thanks`) to this NPC's list at runtime!

### 4.4: Create the Apple Blueprint

Now we need an object the player can pick up to get the "Facts".

1.  Create a new **Blueprint Actor** called `BP_Apple`.
2.  Add a `StaticMeshComponent` and assign an apple mesh.
3.  Add a `BoxCollision` component to detect the player.
4.  In the Event Graph, create a new **Name** variable called `FactToAdd`. Make it **Instance Editable**.
5.  Create the logic to give the player the fact when they overlap the collision box.

The Blueprint graph for this is simple:
<img width="1096" height="778" alt="{45F073A8-6733-4C77-B1E7-980AB52B3722}" src="https://github.com/user-attachments/assets/9d88c9d9-c6ba-4ad0-9fb5-469ed8922c8c" />

```ascii
+-----------------------------+
| (Event) OnActorBeginOverlap |
| (Actor)---------------------+
+-----------------------------+
         |
         v
+-----------------------------+
| Cast To YourPlayerCharacter |
+-----------------------------+
         | (Success)
         v
+-----------------------------+     +--------------------------------+
| Get Game Instance           |-----> Get Subsystem (Dialogue State) |
+-----------------------------+     +--------------------------------+
                                                  |
                                                  v
                                      +-----------------------------+
                                      | Add Fact                    |
                                      | (Fact) <----[Get FactToAdd] |
                                      +-----------------------------+
                                                  |
                                                  v
                                      +-----------------------------+
                                      | DestroyActor (Self)         |
                                      +-----------------------------+
```
FactsToAdd is just variable
<img width="722" height="603" alt="{55537202-5FFD-4262-B92A-217ABA970E54}" src="https://github.com/user-attachments/assets/02af7de4-2df5-4553-b62c-d539d1e8929b" />


6.  Place three `BP_Apple` actors in your level. For each one, select it and go to the **Details** panel. Set the `Fact To Add` variable to `PlayerHasApple1`, `PlayerHasApple2`, and `PlayerHasApple3` respectively.

### 4.5: You're Done!

That's it! Now you can test the full flow:
1.  Talk to the NPC. They will say the `NPC_Start` line.
2.  Choose "Sure, I'll get them for you." The `AcceptAppleQuest` event fires. The `DT_StateChanges` table adds the `PlayerAcceptedAppleQuest` fact and tells the NPC that if the player comes back, they should now start with the `NPC_InProgress` dialogue.
3.  Collect the three apples in the world. Each one adds its unique fact.
4.  Return to the NPC. The system sees `PlayerAcceptedAppleQuest` is true, so dialogue starts at `NPC_InProgress`. The choice to turn in the apples now appears because you have all three apple facts.
5.  Select "Here are the 3 apples." The `CompleteAppleQuest` event fires, cleaning up the old facts and giving the NPC a new "thank you" state for all future conversations.

This example shows how you can build complex, stateful quests by simply defining rules in Data Tables.

