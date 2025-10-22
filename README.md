# Change Key Binding

## Target

Change the Ground Attack input from **Front → Front + Xbox’s X button**  
to **Back → Front + Xbox’s B button**.

## Overall Process

1. **Modify the Preset File**

   Open `DefaultGamepad_Presets`.

   - Add (or locate) the desired button preset.  
     In my case, the file didn’t include the Xbox controller’s **B button** (`Gamepad_FaceButton_Right`) as a trigger for light attacks.
   - Copy the entry for `InputPresets::NewEnumerator6` (the original light attack) from the JSON in this file, and paste it at the **end** of the outer `m_Presets` list.
   - Rename it to `InputPresets::NewEnumerator15` — since the highest value was 14, this new one becomes the 15th.
   - Modify the slot’s key from `Gamepad_FaceButton_Left` to `Gamepad_FaceButton_Right`.
   - Also add `InputPresets::NewEnumerator15` and `Gamepad_FaceButton_Right` to the **Name Map**.
   - Leave the `m_ActionName` of this new preset **unchanged** — it still corresponds to the light attack action.

2. **Edit the Gameplay Mapping**

   Open `DefaultGamepad_Gameplay`.

   - Add `InputPresets::NewEnumerator15` to the **Name Map**.
   - Locate the `m_MappingGroups` section for `InputAction::AttackLightFrontFront`.
   - Inside it, there are two slots:
     - **Slot 1** handles the directional input.
     - **Slot 2** handles the light attack input.
   - In **Slot 1**, change `m_SubInputData` from pointing to `DirFrontFront` (value `-3`) to `DirBackFront` (value `-1`).  
     *(Only modify the Variant — the Value updates automatically.)*
   - In **Slot 2**, change `m_PresetID → m_Value` from `InputPresets::NewEnumerator6` (Xbox’s **X**) to `InputPresets::NewEnumerator15` (Xbox’s **B**).

   That’s it! After repackaging the files with **UnrealPak**, place the `.pak` in the game’s `Mods` folder and test it in-game.

## Reflection

I initially tried modifying the **Generic-type `InputData`** referenced by `m_InputData` in `DefaultGamepad_Gameplay` — even directly editing `AttackLightFrontFront` under `/Inputs` — but none of it had any effect.(However, modifying the file for the generic type did successfully break the action lol.)

No matter how I changed those, the original input logic *(Front, Front, X)* stayed intact.

After rolling back and inspecting `DefaultGamepad_Gameplay` again, I realized that the `m_PresetID` values (`InputPresets::NewEnumerator##`) used inside the slots are actually **defined right next door** in `DefaultGamepad_Presets`.

By comparing these two files side-by-side, I was finally able to trace the entire input flow and fix it.

⚠️ **Note:** 

If you try to force an unrelated preset (e.g. `TakedownValidation2` from `DefaultGamepad_Gameplay`) directly into Slot 2, the game will throw a **Violation error** at startup.

I tried replacing the reference to the original InputData in DefaultGamepad_Gameplay with a copied GenericTypeInputData, but doing so caused the actions to stop working. I haven’t been able to figure out the reason why.