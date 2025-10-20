# Change Key Binding

## Target

Change the Ground Attack input from Front, Front, Xbox’s X button to Back, Front, Xbox’s B button.

## Overall Process

First, in the DefaultGamepad_Presets file, add (or locate) the desired button preset. In my case, the file doesn’t include an Xbox controller B button to trigger a light attack (that is, Gamepad_FaceButton_Right).
Copy the preset for InputPresets::NewEnumerator6 (which is the original light attack) from the JSON in this file, then paste it at the very end of the outer m_Presets list.
Change it to InputPresets::NewEnumerator15 (since there were maximum value is 14, this new one becomes the 15th), and modify the slot’s key from Gamepad_FaceButton_Left to Gamepad_FaceButton_Right.
Remember to also add InputPresets::NewEnumerator15 and Gamepad_FaceButton_Right to the Name Map. As for the m_ActionName of this new preset, leave it as is — it still corresponds to the light attack action, so there’s no need to change it.

Next, open the DefaultGamepad_Gameplay file.
First, add InputPresets::NewEnumerator15 to the NameMap.
Then, find the m_MappingGroups section that belongs to InputAction::AttackLightFrontFront.
Inside it, there are two slots: the first slot handles direction input,the second slot handles the light attack input. Change the m_SubInputData in the first slot from pointing to DirFrontFront (value -3) to DirBackFront (value -1). (You only need to modify the Variant part — the Value will update automatically.)
Next, in the second slot, change m_PresetID → m_Value from InputPresets::NewEnumerator6 (Xbox’s X button) to InputPresets::NewEnumerator15 (Xbox’s B button). That’s it — all done!
After packaging it with UnrealPak, place it in the game’s mods folder and test it out.

## Reflection

I tried several times to modify the Generic-type inputData that the move’s m_InputData in DefaultGamepad_Gameplay points to — and even directly edited AttackLightFrontFront under Inputs — but none of it had any effect.
After multiple attempts, I found that no matter how I changed those two, the original input logic (Front, Front, Xbox’s X) remained unaffected.

After rolling back and going through DefaultGamepad_Gameplay again to figure things out, I discovered that the logic for the PresetID InputPresets::NewEnumerator used in the slots is actually defined right next door in DefaultGamepad_Presets.
By comparing the two files, I managed to outline the overall process.

If you forcefully write an existing button like TakedownValidation2 from DefaultGamepad_Gameplay directly into slot2, the game will trigger a Violation error when launched.
