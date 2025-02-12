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
function selectJob(job)
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

    print("[DEBUG] Minimum player level required for job", job, "is", jobRequirements[job])

    -- Get player identifier
    local playerServerId = GetPlayerServerId(PlayerId())

    -- Trigger event to fetch player's general level (Player Level)
    TriggerEvent('marko_leveling:getPlayerLevel', playerServerId, "Player", function(playerLevel)
        -- Validate if the player level was retrieved
        if not playerLevel then
            print("[ERROR] Failed to retrieve player level")
            QBCore.Functions.Notify("Error retrieving player level", "error", 5000)
            return
        end

        print("[DEBUG] Player general level is", playerLevel)

        -- Check if the player's level is sufficient for the job
        if jobRequirements[job] > playerLevel then
            print("[INFO] Player general level too low for job:", job)
            QBCore.Functions.Notify("Your level is not high enough to start this job", "error", 5000)
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
end

```

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
