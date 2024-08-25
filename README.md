ESX = exports['so_213']:getSharedObject()
local PlayerData = {}

Citizen.CreateThread(function()
    while ESX == nil do
        Citizen.Wait(100)
    end

    PlayerData = ESX.GetPlayerData()
end)

RegisterNetEvent('esx:setJob')
AddEventHandler('esx:setJob', function(job)
    PlayerData.job = job
end)

RegisterNetEvent('sasp:updateDutyStatus')
AddEventHandler('sasp:updateDutyStatus', function(isOnDuty)
    if isOnDuty then
        ESX.ShowNotification(Config.Messages.OnDuty)
    else
        ESX.ShowNotification(Config.Messages.OffDuty)
    end
end)

Citizen.CreateThread(function()
    while true do
        Citizen.Wait(0)
        local playerPed = PlayerPedId()
        local coords = GetEntityCoords(playerPed)
        local isInMarker, currentJob = false, nil

        for _, location in pairs(Config.Locations) do
            local distance = #(coords - location)
            if distance < 10.0 then
                DrawMarker(20, location.x, location.y, location.z - 0.2, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, Config.MarkerSize.x, Config.MarkerSize.y, Config.MarkerSize.z, Config.MarkerColor.r, Config.MarkerColor.g, Config.MarkerColor.b, 100, false, true, 2, nil, nil, false)
                if distance < 1.5 then
                    isInMarker = true
                    for _, job in pairs(Config.Jobs) do
                        if PlayerData.job ~= nil and PlayerData.job.name == job then
                            currentJob = job
                            ESX.ShowHelpNotification(Config.Messages.HelpOnDuty)
                        end
                    end
                end
            end
        end

        if isInMarker and currentJob and IsControlJustReleased(0, Config.Key) then
            TriggerServerEvent('sasp:toggleDuty')
        end

        if not isInMarker then
            Citizen.Wait(500)
        end
    end
end)
