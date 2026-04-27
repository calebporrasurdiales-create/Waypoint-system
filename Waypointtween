-- WaypointSystem v2 (Fully Fixed + Delete + Safe)

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local part = workspace:WaitForChild("MovingPart")

local function setupPlayer(player)
	local mouse = player:GetMouse()

	local waypoints = {}
	local markers = {}
	local lines = {}

	local isMoving = false
	local isPaused = false
	local currentTween = nil
	local currentIndex = 1
	local editMode = false

	-- GUI
	local gui = Instance.new("ScreenGui")
	gui.Name = "WaypointGUI"
	gui.Parent = player:WaitForChild("PlayerGui")

	local frame = Instance.new("Frame", gui)
	frame.Size = UDim2.new(0,220,0,300)
	frame.Position = UDim2.new(0,20,0.5,-150)

	Instance.new("UIListLayout", frame)

	local function btn(text)
		local b = Instance.new("TextButton")
		b.Size = UDim2.new(1,0,0,30)
		b.Text = text
		b.Parent = frame
		return b
	end

	local status = Instance.new("TextLabel", frame)
	status.Size = UDim2.new(1,0,0,40)
	status.BackgroundTransparency = 1

	local edit = btn("Edit")
	local clear = btn("Clear")
	local start = btn("Start")
	local pause = btn("Pause")
	local resume = btn("Resume")

	local function update()
		status.Text = "Points: "..#waypoints.." | Edit: "..tostring(editMode)
	end

	-- ===== VISUALS =====
	local function clearVisuals()
		for _,m in ipairs(markers) do m:Destroy() end
		for _,l in ipairs(lines) do l:Destroy() end
		markers = {}
		lines = {}
	end

	local function drawLines()
		for _,l in ipairs(lines) do l:Destroy() end
		lines = {}

		for i=1,#waypoints-1 do
			local a = waypoints[i]
			local b = waypoints[i+1]

			if typeof(a) ~= "Vector3" or typeof(b) ~= "Vector3" then
				warn("Malformed waypoint (line)", a, b)
				continue
			end

			local p = Instance.new("Part")
			p.Anchored = true
			p.CanCollide = false
			p.Material = Enum.Material.Neon
			p.Color = Color3.fromRGB(0,170,255)

			local dist = (a - b).Magnitude
			p.Size = Vector3.new(0.2,0.2,dist)
			p.CFrame = CFrame.new(a, b) * CFrame.new(0,0,-dist/2)

			p.Parent = workspace
			table.insert(lines,p)
		end
	end

	local function rebuild()
		clearVisuals()
		for _,pos in ipairs(waypoints) do
			if typeof(pos) ~= "Vector3" then
				warn("Malformed waypoint (marker)", pos)
				continue
			end

			local m = Instance.new("Part")
			m.Shape = Enum.PartType.Ball
			m.Size = Vector3.new(1,1,1)
			m.Position = pos
			m.Anchored = true
			m.CanCollide = false
			m.Material = Enum.Material.Neon
			m.Color = Color3.fromRGB(0,255,0)
			m.Parent = workspace
			table.insert(markers,m)
		end
		drawLines()
	end

	-- ===== CLICK SYSTEM =====
	mouse.Button1Down:Connect(function()
		if not editMode then return end
		
		local target = mouse.Target
		
		-- DELETE waypoint
		for i, marker in ipairs(markers) do
			if target == marker then
				table.remove(waypoints, i)
				rebuild()
				update()
				return
			end
		end
		
		-- ADD waypoint
		if not mouse.Hit then return end
		
		local pos = mouse.Hit.Position
		
		if typeof(pos) ~= "Vector3" then
			warn("Malformed waypoint (click)")
			return
		end
		
		table.insert(waypoints, pos)
		rebuild()
		update()
	end)

	-- ===== BUTTONS =====
	edit.MouseButton1Click:Connect(function()
		editMode = not editMode
		update()
	end)

	clear.MouseButton1Click:Connect(function()
		waypoints = {}
		clearVisuals()
		update()
	end)

	-- ===== TWEEN =====
	local function createTween(target)
		if typeof(target) ~= "Vector3" then
			warn("Malformed waypoint (tween)", target)
			return nil
		end
		
		local dist = (part.Position - target).Magnitude
		local t = dist / 10
		
		return TweenService:Create(part, TweenInfo.new(t), {Position = target})
	end

	local function runPath()
		for i=currentIndex,#waypoints do
			if not isMoving then return end
			
			local target = waypoints[i]
			
			if typeof(target) ~= "Vector3" then
				warn("Skipping bad waypoint:", i)
				continue
			end
			
			currentIndex = i
			
			local tw = createTween(target)
			if not tw then continue end
			
			currentTween = tw
			tw:Play()
			
			local done = false
			tw.Completed:Connect(function() done = true end)
			
			while not done do
				if isPaused then
					tw:Cancel()
					repeat task.wait() until not isPaused or not isMoving
					
					if not isMoving then return end
					
					tw = createTween(target)
					if not tw then return end
					
					currentTween = tw
					tw:Play()
					
					done = false
					tw.Completed:Connect(function() done = true end)
				end
				task.wait()
			end
		end
	end

	start.MouseButton1Click:Connect(function()
		if #waypoints > 0 and not isMoving then
			isMoving = true
			isPaused = false
			currentIndex = 1
			
			task.spawn(function()
				runPath()
				isMoving = false
			end)
		end
	end)

	pause.MouseButton1Click:Connect(function()
		isPaused = true
	end)

	resume.MouseButton1Click:Connect(function()
		isPaused = false
	end)

	update()
end

-- run for players
for _,p in ipairs(Players:GetPlayers()) do
	task.spawn(setupPlayer, p)
end

Players.PlayerAdded:Connect(function(p)
	task.spawn(setupPlayer, p)
end)
