# Clio screenshots — Production reference for dummy Clio setup

**Source:** Jared sent these from **production Clio (Kimball Law Inc.)** so you can replicate the same structure in the **dummy Clio** account for the Week 1 demo.

**What to do:** In the dummy Clio account, add practice areas, matter numbering, and the custom field sets listed below. Then in Make.com, when you create a Person and Pending Matter, attach these sets and map GHL/Retell data into them.

---

## Files in this folder

| File | Content | Action in dummy Clio |
|------|---------|----------------------|
| Clio_Practice_Areas_List_1.png, _2.png | List of 23 practice areas | Add same (or subset); e.g. Personal Injury for MVA demo |
| Clio_Practice_Area_Enablement.png | Enabled/Disabled; Personal Injury = Primary | Enable practice areas you need; set PI as primary if doing civil intake |
| Clio_Matter_Numbering.png | Format: year - matter # - client name - first name | Set same matter numbering scheme |
| Clio_Custom_Field_Sets_Table.png | Overview of all field sets | Reference only |
| Clio_Field_Set_A1_General_Intake.png | A1: Narrative notes, Matter subtypes | **Create** this set; map from GHL/Retell |
| Clio_Field_Set_C1_Civil_Update.png | C1: Date of incident, Damages notes, etc. | **Ignore** for demo (per Jared) |
| Clio_Field_Set_C2_Civil_Intake.png | C2: Date/time/location of incident, Seatbelted, Injuries, etc. | **Create** this set; Retell can collect; map to Matter |
| Clio_Field_Set_Z1_All_Clients_Consults.png | Z1: Conflict check, Consult booked, Client retained, etc. | Optional: create if you map these |
| Clio_Field_Set_Z2_How_Did_You_Hear.png | Z2: How did you hear about us (Google, Referral, etc.) | Optional: create if form/Retell captures this |

**Important:** A1 and C2 are the required custom field sets for the demo. When Make.com creates the Matter, it should automatically add these sets (per Jared).
