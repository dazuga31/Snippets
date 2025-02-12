# Integration Guide: Player Level System for Jobs

This guide explains how to integrate the **player level system** into your job selection script, ensuring that players need a **minimum general level** to access jobs.

---

## 1. Modify `config.lua`
Find **`config.lua`** and add the following job level requirements:

```lua
Config.JobRequirements = {
    ["pizza_delivery"] = 1,
    ["news_delivery"] = 1,
    ["mobile_hotdog"] = 1,
    ["forklifter"] = 4,
    ["gardener"] = 4,
    ["trucker"] = 5,
    ["roadhelper"] = 6,
    ["bus_driver"] = 6,
    ["fire_department"] = 8,
    ["hunter"] = 8,
    ["detectorist"] = 5,
    ["project_car"] = 10,
    ["diver"] = 5,
}
```

---

## 2. Update `wais-jobpack/client/editable.lua`
Find the `selectJob` function inside `wais-jobpack/client/editable.lua` and **replace it** with the following updated version:

```lua
RegisterNetEvent("marko_fraction_cloakroom:sendJobSelection")
AddEventHandler("marko_fraction_cloakroom:sendJobSelection", function(job)
    -- Check if the job parameter is provided
    if not job then
        print("[ERROR] selectJob was called without a job argument")
        QBCore.Functions.Notify("Invalid job selection", "error", 5000)
        return
    end

    print("[DEBUG] selectJob called with job:", job)

    -- Retrieve job requirements from config.lua
    local jobRequirements = Config.JobRequirements

    -- Check if the job exists in the job requirements list
    if not jobRequirements[job] then
        print("[ERROR] Job '" .. job .. "' not found in JobRequirements table")
        QBCore.Functions.Notify("This job does not exist", "error", 5000)
        return
    end

    print("[DEBUG] Minimum player level required for job:", job, "is", jobRequirements[job], "| Requesting level from server.")

    -- Request player level from the server
    TriggerServerEvent("marko_fraction_cloakroom:requestPlayerLevelForJob", job)
end)

-- Listen for the response from the server
RegisterNetEvent("marko_fraction_cloakroom:sendPlayerLevelForJob")
AddEventHandler("marko_fraction_cloakroom:sendPlayerLevelForJob", function(job, playerLevel)
    if not playerLevel then
        print("[ERROR] Failed to retrieve player level for job:", job)
        QBCore.Functions.Notify("Error retrieving player level", "error", 5000)
        return
    end

    print("[DEBUG] Player general level is", playerLevel)

    -- Check if the player's level is sufficient for the job
    if playerLevel < Config.JobRequirements[job] then
        print("[INFO] Player general level too low for job:", job, "(Required:", Config.JobRequirements[job], ", Player Level:", playerLevel, ")")
        QBCore.Functions.Notify("Your level (" .. playerLevel .. ") is not high enough to start this job. Required level: " .. Config.JobRequirements[job], "error", 5000)
        return
    end

    -- Assign the job if the level requirement is met
    print("[SUCCESS] Assigning job:", job, "to player")
    QBCore.Functions.Notify("You have successfully started the job: " .. job, "success", 5000)

    if Config.SideJob then
        TriggerEvent('wais:set:sideJob', job)
    else
        TriggerServerEvent('wais:setJob', job)
    end

    -- Set waypoint for the job location
    if Config.Jobs[job] and Config.Jobs[job].menu and Config.Jobs[job].menu.job_menu then
        SetNewWaypoint(Config.Jobs[job].menu.job_menu.x, Config.Jobs[job].menu.job_menu.y)
        print("[DEBUG] Waypoint set for job:", job)
        QBCore.Functions.Notify("A waypoint has been set for your job location", "primary", 5000)
    else
        print("[ERROR] Job location data is missing for:", job)
        QBCore.Functions.Notify("Job location data is missing", "error", 5000)
    end
end)

```

## 3. Add tihs EvenHandler in any Server Side .lua file in wais-jobpack/server

```lua

RegisterNetEvent("marko_fraction_cloakroom:requestPlayerLevelForJob")
AddEventHandler("marko_fraction_cloakroom:requestPlayerLevelForJob", function(job)
    local src = source  -- Get the player's server ID
    local role = "Player"  -- Specify the role for which to retrieve level

    print("[DEBUG] Requesting player level for source:", src, "Role:", role, "Job:", job)

    -- Fetch the player's level using the exported function from marko_leveling
    exports['marko_leveling']:getPlayerLevel(src, role, function(playerLevel)
        if playerLevel then
            print("[DEBUG] Player Level Retrieved:", playerLevel)
        else
            print("[ERROR] Failed to retrieve player level")
        end

        -- Send the level back to the client
        TriggerClientEvent("marko_fraction_cloakroom:sendPlayerLevelForJob", src, job, playerLevel)
    end)
end)
```

---

---

## 3. Logic Explanation
- **General player level (`Player Level`) is now used** instead of job-specific skill levels.
- The function checks if the player has **enough general level** to start a job.
- Players **unlock higher-level jobs** as they level up.
- Debug messages (`[DEBUG]`, `[ERROR]`, `[INFO]`, `[SUCCESS]`) are included for troubleshooting.

---

## 📌 Need Help?
Join our Discord for support:  
🔗 **[https://discord.gg/caJ3aNae](https://discord.gg/caJ3aNae)** 🚀
