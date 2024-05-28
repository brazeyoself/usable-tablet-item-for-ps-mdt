# usable-tablet-item-for-ps-mdt
create a tablet item for ps-mdt

 - image for inventory:
![mdttablet](https://github.com/brazeyoself/usable-tablet-item-for-ps-mdt/assets/88394932/63c82373-3a93-46f3-8e38-19256c12435f)

• First go to qb-core/shared/items and add the following:
```
mdttablet                    = { name = 'mdttablet', label = 'MD Tablet', weight = 500, type = 'item', image = 'mdttablet.png', unique = false, useable = true, shouldClose = true, combinable = nil, description = 'Mobile Datenbank des Staates' },
```
• I assume you are using the pictures from qb-inventory, which has a tablet picture in it, otherwise you have to create one

• next go to ps-mdt/server/main.lua on line 230:
```
RegisterNetEvent('mdt:server:openMDT', function()
```
• Above that you copy the following:
```
QBCore.Functions.CreateUseableItem('mdttablet', function(source)
	TriggerClientEvent('mdt:client:Tablet', source)
end)
```

• then go to ps-mdt/client/main.lua to line 89:
```
RegisterKeyMapping('mdt', 'Open Police MDT', 'keyboard', 'k')
````
• and comment it out or delete like here
```
--RegisterKeyMapping('mdt', 'Open Police MDT', 'keyboard', 'k')
```
• go to line 91 to 106 and comment or delete the lines like here:
```
--[[RegisterCommand('mdt', function()
    local plyPed = PlayerPedId()
    PlayerData = QBCore.Functions.GetPlayerData()
    if not PlayerData.metadata["isdead"] and not PlayerData.metadata["inlaststand"] and not PlayerData.metadata["ishandcuffed"] and not IsPauseMenuActive() then
        if GetJobType(PlayerData.job.name) ~= nil then
            TriggerServerEvent('mdt:server:openMDT')
            TriggerServerEvent('mdt:requestOfficerData')
        end
    else
        QBCore.Functions.Notify("Can't do that!", "error")
    end
end, false)

Citizen.CreateThread(function()
    TriggerEvent('chat:addSuggestion', '/mdt', 'Open the emergency services MDT', {})
end)]]
```
• and just below you copy the following:
```
RegisterNetEvent('mdt:client:Tablet', function()
    local plyPed = PlayerPedId()
    PlayerData = QBCore.Functions.GetPlayerData()
    if not PlayerData.metadata["isdead"] and not PlayerData.metadata["inlaststand"] and not PlayerData.metadata["ishandcuffed"] and not IsPauseMenuActive() then
        if GetJobType(PlayerData.job.name) ~= nil then
            openmdtanim(true)
            TriggerServerEvent('mdt:server:openMDT')
            TriggerServerEvent('mdt:requestOfficerData')
        end
    else
        QBCore.Functions.Notify("Cant do that!", "error")
    end
end, false)

local tablet = false
local tabletDict = "amb@code_human_in_bus_passenger_idles@female@tablet@base"
local tabletAnim = "base"
local tabletProp = "prop_cs_tablet"
local tabletBone = 60309
local tabletOffset = vector3(0.03, 0.002, -0.0)
local tabletRot = vector3(10.0, 160.0, 0.0)



function openmdtanim(toggle)
    print("mdt opening")
    if toggle and not tablet then
        tablet = true

        if not IsPedInAnyVehicle(PlayerPedId(), false) then
            Citizen.CreateThread(function()
                RequestAnimDict(tabletDict)

                while not HasAnimDictLoaded(tabletDict) do
                    Citizen.Wait(150)
                end

                RequestModel(tabletProp)

                while not HasModelLoaded(tabletProp) do
                    Citizen.Wait(150)
                end

                local playerPed = PlayerPedId()
                local tabletObj = CreateObject(tabletProp, 0.0, 0.0, 0.0, true, true, false)
                local tabletBoneIndex = GetPedBoneIndex(playerPed, tabletBone)

                SetCurrentPedWeapon(playerPed, weapon_unarmed, true)
                AttachEntityToEntity(tabletObj, playerPed, tabletBoneIndex, tabletOffset.x, tabletOffset.y, tabletOffset.z, tabletRot.x, tabletRot.y, tabletRot.z, true, false, false, false, 2, true)
                SetModelAsNoLongerNeeded(tabletProp)

                while tablet do
                    Citizen.Wait(100)
                    playerPed = PlayerPedId()

                    if not IsEntityPlayingAnim(playerPed, tabletDict, tabletAnim, 3) then
                        TaskPlayAnim(playerPed, tabletDict, tabletAnim, 3.0, 3.0, -1, 49, 0, 0, 0, 0)
                    end
                end

                ClearPedSecondaryTask(playerPed)

                Citizen.Wait(450)

                DetachEntity(tabletObj, true, false)
                DeleteEntity(tabletObj)
            end)
        end
    elseif not toggle and tablet then
        tablet = false
    end
end
```
In this way you can only open the mdt if you are using a tablet, the commands like /mdt or "K" are deactivated


