# DMP COMMAND

Source-controlled working copy for the DMP COMMAND system (Power Automate flows + Power App).

## Structure

- `PowerAutomate/DMP_COMMAND_Solution/` - Dataverse solution source (unpacked via `pac solution unpack`).
  `Source/Workflows/*.json` contains the full definition of every agent flow. New agents are added here
  automatically as new `.json` files once added to the `DMP_COMMAND_Solution` solution in Power Apps and
  re-exported - no structural changes needed to accommodate additional agents.
- `PowerApp/DMP_COMMAND/` - Canvas app source (unpacked via `pac canvas unpack`).
  `Source/Src/*.pa.yaml` contains the screen/component definitions.
- `Documentation/` - Mission & AI working rules, backlog, and periodic exports of the two SharePoint lists
  (`DMP Command Configuration`, `DMP Command Agent Status`) for reference. These CSVs are point-in-time
  snapshots, not the live data source.

## Workflow (Power Automate side)

```
pac solution export --name "DMP_COMMAND_Solution" --path bin\DMP_COMMAND_Solution.zip --overwrite true
pac solution unpack --zipfile bin\DMP_COMMAND_Solution.zip --folder PowerAutomate\DMP_COMMAND_Solution\Source --packagetype Unmanaged
# edit PowerAutomate\DMP_COMMAND_Solution\Source\Workflows\*.json directly
pac solution pack --zipfile bin\DMP_COMMAND_Solution.zip --folder PowerAutomate\DMP_COMMAND_Solution\Source --packagetype Unmanaged
pac solution import --path bin\DMP_COMMAND_Solution.zip
```

## Workflow (Power App side)

```
pac canvas download --name "DMP COMMAND" --file-name DMP_COMMAND.msapp --overwrite
pac canvas unpack --msapp DMP_COMMAND.msapp --sources PowerApp\DMP_COMMAND\Source --layout SourceCode --overwrite
# edit PowerApp\DMP_COMMAND\Source\Src\*.pa.yaml directly
pac canvas pack --sources PowerApp\DMP_COMMAND\Source --msapp DMP_COMMAND_TEST.msapp --overwrite
# import DMP_COMMAND_TEST.msapp via Power Apps portal > Apps > Import app > From file
```

Built binaries (`.msapp`, solution `.zip`) are not tracked in this repo - only human-readable source.
