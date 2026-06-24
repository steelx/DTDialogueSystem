# 💬 DTDialogueSystem - Quick Start Guide

Welcome to DTDialogueSystem, a robust, data-driven, and highly scalable dialogue plugin for Unreal Engine 5. This system utilizes native FGameplayTag architecture for memory-safe state tracking and branching narratives.
[![Watch the video](https://www.youtube.com/watch?v=aPGQqGVAPp8)](https://www.youtube.com/watch?v=aPGQqGVAPp8)


Follow this guide to get your first conversation running in minutes.

### ⚙️ Step 1: Enable Gameplay Tags

This plugin relies heavily on Unreal Engine's native Gameplay Tags for absolute type safety.
> you can name your tags anything, it doesnt matter.
<img width="2096" height="738" alt="image" src="https://github.com/user-attachments/assets/a83fe5c4-636e-41f9-8cbc-a03a5dc84996" />


    Open your project in Unreal Engine.

    Go to Edit -> Project Settings -> Project -> Gameplay Tags.

    Add your narrative tags to the dictionary. (Example: Dialogue.NPC.Start, State.Quest.Apple.Accepted).

### 🗄️ Step 2: Create the Data Tables

<img width="706" height="378" alt="image" src="https://github.com/user-attachments/assets/091edc0f-529b-4a7d-b134-5be83bf30c1a" />

Your dialogue is driven entirely by Data Tables. You will need to create three separate tables:

    - Right-click in the Content Browser -> Miscellaneous -> Data Table.

    - Create the following tables using the included structs:

        DT_DialogueLines: Select `DialogueLine` as the row structure.

        DT_PlayerChoices: Select `PlayerChoice` as the row structure.

        DT_StateChanges: Select `DialogueStateChange` as the row structure.

<img width="2554" height="1231" alt="image" src="https://github.com/user-attachments/assets/8f570aa3-8fdc-4d91-a567-da6d70e76082" />


⚠️ CRITICAL RULE: The Row Name of your Data Table items must exactly match your Gameplay Tag string. For example, if your starting line tag is Dialogue.NPC.Start, the Row Name in DT_DialogueLines must be exactly Dialogue.NPC.Start.

### 🔌 Step 3: Configure Project Settings

Now we tell the global Dialogue Manager where to find your data.
<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/cc8b839e-cac9-48d1-bd3b-6c29d3d31c81" />

    Go to Edit -> Project Settings -> Game -> Dialogue Settings.

    Assign the three Data Tables you just created to their respective slots.

    Under Dialogue UI, select your custom Dialogue Widget (e.g., WBP_DialogueFooter).

### 🧠 Step 4: Give Your Actors "Memory"

For the system to track who is talking and remember where the conversation left off, participants need a memory component.
<img width="2560" height="1389" alt="image" src="https://github.com/user-attachments/assets/d60521fd-99ab-4c4b-adcd-cb2b2fe9c6bd" />


    Open your Player Character Blueprint.

    Add the DTDialogueParticipant component. Set the Participant Name (e.g., "Player").

    Open your NPC Blueprint.

    Add the DTDialogueParticipant component. Set their Participant Name.

    On the NPC's component, set the Current Dialogue ID to your starting tag (e.g., Dialogue.NPC.Start).

### 🎬 Step 5: Trigger the Dialogue

Starting a conversation is as simple as calling a single node.
<img width="1375" height="615" alt="image" src="https://github.com/user-attachments/assets/ace309d5-285b-493f-a070-f61c4f5bacb5" />


    In your Player Character (or Interaction component), create an input event (e.g., E key).

    Perform a Line Trace or Sphere Trace to detect the NPC.

    If hit, call the Start Dialogue node.

        Initiator: Plug in Self (The Player).

        Target NPC: Plug in the Hit Actor (The NPC).

> The system will automatically pause game input, slide your UI onto the screen, and begin the typewriter effect!

### 📡 Bonus: Listening to Dialogue Events

When a player makes a choice that fires an Event Tag (e.g., State.Quest.Completed), you can easily listen for this globally to update your game world, grant XP, or give items.
<img width="1587" height="755" alt="image" src="https://github.com/user-attachments/assets/8ad04517-47e7-45bc-b3e6-303dbba4db69" />


    Anywhere in your Blueprints (GameMode, Player, UI), call Get Dialogue Subsystem.

    Drag off the return value and call Bind Event to On Dialogue Event Fired.

    Create a Custom Event.

    Use a Matches Tag node on the output EventTag to check if it matches the specific event you are waiting for.

### 🐛 Debugging

Need to see what tags the player currently has?
While playing in the editor, press the tilde key (~) to open the console and type:
Dialogue.DebugTags

This will print all active player facts and their exact integer quantities directly to the screen!
