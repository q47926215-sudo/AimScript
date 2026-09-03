--========================================================
-- VECTOR AIM v2 + ESP
-- Advanced LocalScript
-- StarterPlayer > StarterPlayerScripts
--========================================================

--========================================================
-- SERVICES
--========================================================

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

--========================================================
-- PLAYER / CAMERA
--========================================================

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = Workspace.CurrentCamera

--========================================================
-- AIM CONFIG
--========================================================

local AIM_ENABLED = false
local AIM_KEY = Enum.KeyCode.Q
local AIM_MODE = "Hold"

local FOV_RADIUS = 180
local STICKY_FOV = 240

local AIM_DISTANCE = 1000

local SMOOTHNESS = 0.18
local AIM_SMOOTH_MODE = "Exponential"

local TARGET_PLAYERS = true
local TARGET_NPCS = true

local TARGET_PART = "Head"

local TEAM_CHECK = false
local WALL_CHECK = false
local IGNORE_DEAD = true
local IGNORE_FRIENDS = false

local TARGET_LOCK = true
local STICKY_AIM = true

local TARGET_PRIORITY = "Crosshair"

local PRIORITY_CROSSHAIR_WEIGHT = 70
local PRIORITY_DISTANCE_WEIGHT = 30
local PRIORITY_HEALTH_WEIGHT = 0

local PREDICTION_ENABLED = false
local AUTO_PREDICTION = true
local PREDICTION_TIME = 0.12

local PREDICTION_X = 1
local PREDICTION_Y = 1
local PREDICTION_Z = 1

local WALL_CHECK_MODE = "Smart"

local SHOW_FOV = true
local SHOW_TARGET_LINE = true
local SHOW_TARGET_DOT = true
local SHOW_TARGET_INFO = true
local SHOW_PREDICTION_POINT = true

local TARGET_BOX = true

local REACTION_DELAY = 0

local MAX_TURN_SPEED = 1000

local AIM_DEADZONE = 0

--========================================================
-- ESP CONFIG
--========================================================

local ESP_ENABLED = true
local PLAYERS_ENABLED = true
local NPC_ENABLED = true

local PLAYER_COLOR = Color3.fromRGB(65, 255, 150)
local NPC_COLOR = Color3.fromRGB(255, 75, 95)

local ESP_FILL_TRANSPARENCY = 0.70
local ESP_OUTLINE_TRANSPARENCY = 0

local ESP_NAMES = true
local ESP_DISTANCE = true
local ESP_HEALTH = true

--========================================================
-- STATE
--========================================================

local currentTarget = nil
local currentTargetCharacter = nil

local targetAcquiredTime = 0

local listeningForKey = false
local menuDestroyed = false
local menuMinimized = false

local menuPage = "AIM"

local aimHeld = false

local targetCandidates = {}

local lastCandidateScan = 0
local candidateScanInterval = 0.20

local debugVisible = false

--========================================================
-- PROFILE DATA
--========================================================

local Profiles = {
	LEGIT = {
		FOV_RADIUS = 100,
		STICKY_FOV = 130,
		AIM_DISTANCE = 500,
		SMOOTHNESS = 0.08,
		PREDICTION_ENABLED = true,
		AUTO_PREDICTION = true,
		PREDICTION_TIME = 0.10,
		TARGET_PRIORITY = "Crosshair",
		TARGET_PART = "Head",
		WALL_CHECK = true,
		TARGET_LOCK = true,
		STICKY_AIM = true
	},

	RAGE = {
		FOV_RADIUS = 360,
		STICKY_FOV = 430,
		AIM_DISTANCE = 2000,
		SMOOTHNESS = 0.75,
		PREDICTION_ENABLED = true,
		AUTO_PREDICTION = true,
		PREDICTION_TIME = 0.16,
		TARGET_PRIORITY = "Crosshair",
		TARGET_PART = "Head",
		WALL_CHECK = false,
		TARGET_LOCK = true,
		STICKY_AIM = true
	},

	NPC = {
		FOV_RADIUS = 250,
		STICKY_FOV = 300,
		AIM_DISTANCE = 900,
		SMOOTHNESS = 0.25,
		PREDICTION_ENABLED = false,
		AUTO_PREDICTION = false,
		PREDICTION_TIME = 0.08,
		TARGET_PRIORITY = "Distance",
		TARGET_PART = "UpperTorso",
		WALL_CHECK = true,
		TARGET_LOCK = true,
		STICKY_AIM = true
	}
}

--========================================================
-- UTILITY
--========================================================

local function clamp01(value)
	return math.clamp(value, 0, 1)
end

local function safeDestroy(object)
	if object and object.Parent then
		object:Destroy()
	end
end

--========================================================
-- ESP SYSTEM
--========================================================

local function removeESP(model)
	if not model then
		return
	end

	local esp = model:FindFirstChild("LocalESP")

	if esp then
		esp:Destroy()
	end

	local billboard = model:FindFirstChild("LocalESP_Info")

	if billboard then
		billboard:Destroy()
	end
end

local function createESPInfo(model, color)
	if not model then
		return
	end

	if not ESP_NAMES
		and not ESP_DISTANCE
		and not ESP_HEALTH then
		return
	end

	if model:FindFirstChild("LocalESP_Info") then
		return
	end

	local head = model:FindFirstChild("Head")
		or model:FindFirstChild("HumanoidRootPart")

	if not head then
		return
	end

	local billboard = Instance.new("BillboardGui")

	billboard.Name = "LocalESP_Info"
	billboard.Adornee = head
	billboard.Size = UDim2.fromOffset(190, 55)
	billboard.StudsOffset = Vector3.new(0, 3.2, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = model

	local background = Instance.new("Frame")

	background.Size = UDim2.fromScale(1, 1)
	background.BackgroundColor3 = Color3.fromRGB(10, 13, 18)
	background.BackgroundTransparency = 0.25
	background.BorderSizePixel = 0
	background.Parent = billboard

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 7)
	corner.Parent = background

	local stroke = Instance.new("UIStroke")
	stroke.Color = color
	stroke.Thickness = 1
	stroke.Transparency = 0.35
	stroke.Parent = background

	local text = Instance.new("TextLabel")

	text.Name = "Info"
	text.Size = UDim2.new(1, -10, 1, -6)
	text.Position = UDim2.fromOffset(5, 3)

	text.BackgroundTransparency = 1

	text.TextColor3 = color
	text.TextSize = 11
	text.Font = Enum.Font.GothamBold

	text.TextXAlignment = Enum.TextXAlignment.Left
	text.TextYAlignment = Enum.TextYAlignment.Top

	text.Parent = background

	local humanoid = model:FindFirstChildOfClass("Humanoid")

	task.spawn(function()
		while billboard.Parent and model.Parent do

			local lines = {}

			if ESP_NAMES then
				local player = nil

				for _, p in ipairs(Players:GetPlayers()) do
					if p.Character == model then
						player = p
						break
					end
				end

				if player then
					table.insert(lines, player.DisplayName)
				else
					table.insert(lines, "NPC / BOT")
				end
			end

			if ESP_DISTANCE then
				if LocalPlayer.Character then

					local myRoot =
						LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

					local targetRoot =
						model:FindFirstChild("HumanoidRootPart")

					if myRoot and targetRoot then

						local distance =
							(myRoot.Position - targetRoot.Position).Magnitude

						table.insert(
							lines,
							"Distance: "
								.. math.floor(distance)
								.. " studs"
						)
					end
				end
			end

			if ESP_HEALTH and humanoid then

				table.insert(
					lines,
					"HP: "
						.. math.floor(humanoid.Health)
						.. " / "
						.. math.floor(humanoid.MaxHealth)
				)

			end

			text.Text = table.concat(lines, "\n")

			task.wait(0.12)
		end
	end)
end

local function createESP(model, color)
	if not model or not model:IsA("Model") then
		return
	end

	local humanoid = model:FindFirstChildOfClass("Humanoid")

	if not humanoid then
		return
	end

	if model:FindFirstChild("LocalESP") then
		return
	end

	local highlight = Instance.new("Highlight")

	highlight.Name = "LocalESP"
	highlight.Adornee = model

	highlight.FillColor = color
	highlight.OutlineColor = color

	highlight.FillTransparency =
		ESP_FILL_TRANSPARENCY

	highlight.OutlineTransparency =
		ESP_OUTLINE_TRANSPARENCY

	highlight.DepthMode =
		Enum.HighlightDepthMode.AlwaysOnTop

	highlight.Parent = model

	createESPInfo(model, color)
end

local function isPlayerCharacter(model)
	for _, player in ipairs(Players:GetPlayers()) do
		if player.Character == model then
			return true
		end
	end

	return false
end

local function updateModelESP(model)
	if not model or not model:IsA("Model") then
		return
	end

	local humanoid =
		model:FindFirstChildOfClass("Humanoid")

	if not humanoid then
		return
	end

	if not ESP_ENABLED then

		removeESP(model)
		return

	end

	if isPlayerCharacter(model) then

		if PLAYERS_ENABLED then
			createESP(model, PLAYER_COLOR)
		else
			removeESP(model)
		end

	else

		if NPC_ENABLED then
			createESP(model, NPC_COLOR)
		else
			removeESP(model)
		end

	end
end

local function updateAllESP()
	for _, object in ipairs(
		Workspace:GetDescendants()
	) do

		if object:IsA("Model")
			and object:FindFirstChildOfClass("Humanoid") then

			updateModelESP(object)

		end
	end
end

--========================================================
-- AIM HELPERS
--========================================================

local function getCharacterHumanoid(character)
	if not character then
		return nil
	end

	return character:FindFirstChildOfClass("Humanoid")
end

local function isAlive(character)
	local humanoid = getCharacterHumanoid(character)

	if not humanoid then
		return false
	end

	return humanoid.Health > 0
end

local function getPlayerFromCharacter(model)
	for _, player in ipairs(Players:GetPlayers()) do
		if player.Character == model then
			return player
		end
	end

	return nil
end

local function isFriend(player)
	if not player then
		return false
	end

	local success, result = pcall(function()
		return LocalPlayer:IsFriendsWith(player.UserId)
	end)

	return success and result
end

local function allowedPlayer(player)
	if not player then
		return false
	end

	if player == LocalPlayer then
		return false
	end

	if TEAM_CHECK then

		if LocalPlayer.Team
			and player.Team
			and LocalPlayer.Team == player.Team then

			return false
		end
	end

	if IGNORE_FRIENDS then
		if isFriend(player) then
			return false
		end
	end

	return true
end

--========================================================
-- TARGET PART
--========================================================

local TARGET_PARTS = {
	"Head",
	"UpperTorso",
	"HumanoidRootPart"
}

local function getTargetPart(character)
	if not character then
		return nil
	end

	if TARGET_PART == "Head" then

		return character:FindFirstChild("Head")
			or character:FindFirstChild("UpperTorso")
			or character:FindFirstChild("HumanoidRootPart")

	elseif TARGET_PART == "UpperTorso" then

		return character:FindFirstChild("UpperTorso")
			or character:FindFirstChild("Torso")
			or character:FindFirstChild("HumanoidRootPart")

	elseif TARGET_PART == "HumanoidRootPart" then

		return character:FindFirstChild("HumanoidRootPart")

	end

	return character:FindFirstChild("Head")
end

--========================================================
-- SMART TARGET POINTS
--========================================================

local function getVisiblePoints(character, mainPart)
	if not character or not mainPart then
		return {}
	end

	local points = {}

	table.insert(points, mainPart)

	local head = character:FindFirstChild("Head")
	local upperTorso = character:FindFirstChild("UpperTorso")
	local root = character:FindFirstChild("HumanoidRootPart")

	if head then
		table.insert(points, head)
	end

	if upperTorso then
		table.insert(points, upperTorso)
	end

	if root then
		table.insert(points, root)
	end

	return points
end

--========================================================
-- RAYCAST
--========================================================

local rayParams = RaycastParams.new()

rayParams.FilterType =
	Enum.RaycastFilterType.Exclude

rayParams.IgnoreWater = true

local function simpleLineOfSight(part)
	local character =
		LocalPlayer.Character

	if not character then
		return false
	end

	rayParams.FilterDescendantsInstances = {
		character
	}

	local origin =
		Camera.CFrame.Position

	local direction =
		part.Position - origin

	local result =
		Workspace:Raycast(
			origin,
			direction,
			rayParams
		)

	if not result then
		return true
	end

	return result.Instance:IsDescendantOf(
		part.Parent
	)
end

local function smartLineOfSight(character, mainPart)

	local points =
		getVisiblePoints(
			character,
			mainPart
		)

	for _, point in ipairs(points) do

		if simpleLineOfSight(point) then
			return true
		end

	end

	return false
end

local function hasLineOfSight(character, part)

	if not WALL_CHECK then
		return true
	end

	if WALL_CHECK_MODE == "Simple" then
		return simpleLineOfSight(part)
	end

	if WALL_CHECK_MODE == "Smart" then
		return smartLineOfSight(
			character,
			part
		)
	end

	if WALL_CHECK_MODE == "Strict" then

		return simpleLineOfSight(part)

	end

	return true
end

--========================================================
-- PREDICTION
--========================================================

local function getVelocity(part)

	if not part then
		return Vector3.zero
	end

	return part.AssemblyLinearVelocity
end

local function calculatePredictionTime(distance)

	if not AUTO_PREDICTION then
		return PREDICTION_TIME
	end

	-- Автоматическое увеличение
	-- prediction с дистанцией

	local base =
		0.04 + (distance / 1200) * 0.16

	return math.clamp(
		base,
		0.04,
		0.35
	)
end

local function getPredictedPosition(part)

	if not PREDICTION_ENABLED then
		return part.Position
	end

	local distance =
		(
			Camera.CFrame.Position
			- part.Position
		).Magnitude

	local predictionTime =
		calculatePredictionTime(distance)

	local velocity =
		getVelocity(part)

	local prediction =
		Vector3.new(
			velocity.X * predictionTime * PREDICTION_X,
			velocity.Y * predictionTime * PREDICTION_Y,
			velocity.Z * predictionTime * PREDICTION_Z
		)

	return part.Position + prediction
end

--========================================================
-- TARGET CACHE
--========================================================

local function scanTargets()

	targetCandidates = {}

	-- PLAYERS
	if TARGET_PLAYERS then

		for _, player in ipairs(
			Players:GetPlayers()
		) do

			if allowedPlayer(player)
				and player.Character
				and isAlive(player.Character) then

				local part =
					getTargetPart(player.Character)

				if part then

					table.insert(
						targetCandidates,
						{
							Character = player.Character,
							Part = part,
							Player = player,
							IsNPC = false
						}
					)

				end

			end

		end

	end

	-- NPC
	if TARGET_NPCS then

		for _, object in ipairs(
			Workspace:GetChildren()
		) do

			if object:IsA("Model")
				and object ~= LocalPlayer.Character
				and not isPlayerCharacter(object)
				and object:FindFirstChildOfClass("Humanoid")
				and isAlive(object) then

				local part =
					getTargetPart(object)

				if part then

					table.insert(
						targetCandidates,
						{
							Character = object,
							Part = part,
							Player = nil,
							IsNPC = true
						}
					)

				end

			end

		end

	end
end

--========================================================
-- TARGET SCORING
--========================================================

local function getTargetScreenData(part)

	local screenPosition, visible =
		Camera:WorldToViewportPoint(
			part.Position
		)

	return screenPosition, visible
end

local function getDistanceToTarget(part)

	return (
		Camera.CFrame.Position
		- part.Position
	).Magnitude

end

local function getScreenDistance(part)

	local mouse =
		UserInputService:GetMouseLocation()

	local screenPosition, visible =
		getTargetScreenData(part)

	if not visible or screenPosition.Z <= 0 then
		return math.huge
	end

	return (
		Vector2.new(
			screenPosition.X,
			screenPosition.Y
		) - mouse
	).Magnitude
end

local function getPriorityScore(data)

	local part = data.Part

	if not part then
		return math.huge
	end

	local screenDistance =
		getScreenDistance(part)

	if screenDistance == math.huge then
		return math.huge
	end

	local worldDistance =
		getDistanceToTarget(part)

	local humanoid =
		getCharacterHumanoid(
			data.Character
		)

	local healthRatio = 1

	if humanoid then

		if humanoid.MaxHealth > 0 then

			healthRatio =
				humanoid.Health
				/ humanoid.MaxHealth

		end

	end

	-- NORMALIZED VALUES
	local crosshairScore =
		math.clamp(
			screenDistance
			/ math.max(FOV_RADIUS, 1),
			0,
			10
		)

	local distanceScore =
		math.clamp(
			worldDistance
			/ math.max(AIM_DISTANCE, 1),
			0,
			10
		)

	local healthScore =
		healthRatio * 10

	if TARGET_PRIORITY == "Crosshair" then

		return crosshairScore
			+ distanceScore * 0.05

	elseif TARGET_PRIORITY == "Distance" then

		return distanceScore
			+ crosshairScore * 0.05

	elseif TARGET_PRIORITY == "Health" then

		return healthScore
			+ crosshairScore * 0.05

	elseif TARGET_PRIORITY == "Threat" then

		return
			distanceScore * 0.55
			+ crosshairScore * 0.30
			+ healthScore * 0.15

	elseif TARGET_PRIORITY == "Custom" then

		local totalWeight =
			PRIORITY_CROSSHAIR_WEIGHT
			+ PRIORITY_DISTANCE_WEIGHT
			+ PRIORITY_HEALTH_WEIGHT

		if totalWeight <= 0 then
			return crosshairScore
		end

		return
			(
				crosshairScore
				* PRIORITY_CROSSHAIR_WEIGHT
				+
				distanceScore
				* PRIORITY_DISTANCE_WEIGHT
				+
				healthScore
				* PRIORITY_HEALTH_WEIGHT
			)
			/ totalWeight

	end

	return crosshairScore
end

--========================================================
-- BEST TARGET
--========================================================

local function getBestTarget()

	local best = nil
	local bestScore = math.huge

	for _, data in ipairs(
		targetCandidates
	) do

		if data.Character
			and data.Part
			and data.Part.Parent then

			if IGNORE_DEAD
				and not isAlive(data.Character) then

				continue

			end

			local worldDistance =
				getDistanceToTarget(
					data.Part
				)

			if worldDistance > AIM_DISTANCE then
				continue
			end

			local screenDistance =
				getScreenDistance(
					data.Part
				)

			if screenDistance == math.huge then
				continue
			end

			if screenDistance > FOV_RADIUS then
				continue
			end

			if not hasLineOfSight(
				data.Character,
				data.Part
			) then

				continue

			end

			local score =
				getPriorityScore(data)

			if score < bestScore then

				bestScore = score
				best = data

			end

		end

	end

	return best, bestScore
end

--========================================================
-- TARGET LOCK VALIDATION
--========================================================

local function isCurrentTargetValid()

	if not currentTarget then
		return false
	end

	if not currentTarget.Parent then
		return false
	end

	local character =
		currentTarget.Parent

	if not isAlive(character) then
		return false
	end

	local distance =
		getDistanceToTarget(
			currentTarget
		)

	if distance > AIM_DISTANCE then
		return false
	end

	local screenDistance =
		getScreenDistance(
			currentTarget
		)

	if screenDistance == math.huge then
		return false
	end

	local maximumFOV =
		STICKY_AIM
		and STICKY_FOV
		or FOV_RADIUS

	if screenDistance > maximumFOV then
		return false
	end

	if WALL_CHECK then

		if not hasLineOfSight(
			character,
			currentTarget
		) then

			return false

		end

	end

	return true
end

--========================================================
-- TARGET ACQUISITION
--========================================================

local function acquireTarget()

	local best =
		getBestTarget()

	if not best then

		currentTarget = nil
		currentTargetCharacter = nil

		return

	end

	currentTarget =
		best.Part

	currentTargetCharacter =
		best.Character

	targetAcquiredTime =
		os.clock()
end

local function releaseTarget()

	currentTarget = nil
	currentTargetCharacter = nil

end

--========================================================
-- SMOOTHING
--========================================================

local function calculateSmooth(dt)

	local smooth =
		math.clamp(
			SMOOTHNESS,
			0.001,
			1
		)

	if AIM_SMOOTH_MODE == "Linear" then

		return smooth

	elseif AIM_SMOOTH_MODE == "Exponential" then

		return 1 - math.pow(
			1 - smooth,
			dt * 60
		)

	elseif AIM_SMOOTH_MODE == "Fast" then

		return math.clamp(
			smooth * dt * 90,
			0,
			1
		)

	elseif AIM_SMOOTH_MODE == "Slow" then

		return math.clamp(
			smooth * dt * 30,
			0,
			1
		)

	elseif AIM_SMOOTH_MODE == "Spring" then

		return math.clamp(
			1 - math.exp(
				-smooth * dt * 15
			),
			0,
			1
		)

	end

	return smooth
end

--========================================================
-- AIM MOVEMENT
--========================================================

local function aimAtPosition(position, dt)

	local cameraPosition =
		Camera.CFrame.Position

	local direction =
		position - cameraPosition

	if direction.Magnitude <= 0.001 then
		return
	end

	direction =
		direction.Unit

	-- Deadzone
	if AIM_DEADZONE > 0 then

		local mouse =
			UserInputService:GetMouseLocation()

		local screenPosition =
			Camera:WorldToViewportPoint(
				position
			)

		if screenPosition.Z > 0 then

			local distance =
				(
					Vector2.new(
						screenPosition.X,
						screenPosition.Y
					)
					- mouse
				).Magnitude

			if distance <= AIM_DEADZONE then
				return
			end

		end

	end

	local targetCFrame =
		CFrame.lookAt(
			cameraPosition,
			cameraPosition + direction
		)

	local alpha =
		calculateSmooth(dt)

	Camera.CFrame =
		Camera.CFrame:Lerp(
			targetCFrame,
			alpha
		)
end

local function aimAtTarget(part, dt)

	if not part then
		return
	end

	if not part.Parent then
		return
	end

	local position =
		getPredictedPosition(part)

	aimAtPosition(
		position,
		dt
	)

end

--========================================================
-- FOV GUI
--========================================================

local FOVGui = Instance.new("ScreenGui")

FOVGui.Name = "VectorFOV"
FOVGui.ResetOnSpawn = false
FOVGui.IgnoreGuiInset = true
FOVGui.DisplayOrder = 998
FOVGui.Parent = PlayerGui

local FOVCircle = Instance.new("Frame")

FOVCircle.Name = "FOVCircle"

FOVCircle.AnchorPoint =
	Vector2.new(0.5, 0.5)

FOVCircle.BackgroundTransparency = 1

FOVCircle.Parent = FOVGui

createCorner = function(parent, radius)

	local corner =
		Instance.new("UICorner")

	corner.CornerRadius =
		UDim.new(0, radius)

	corner.Parent = parent

	return corner

end

local FOVCorner =
	createCorner(
		FOVCircle,
		500
	)

local FOVStroke =
	Instance.new("UIStroke")

FOVStroke.Color =
	Color3.fromRGB(
		80,
		220,
		255
	)

FOVStroke.Thickness = 1.5
FOVStroke.Transparency = 0.12

FOVStroke.Parent =
	FOVCircle

--========================================================
-- STICKY FOV
--========================================================

local StickyFOVCircle =
	FOVCircle:Clone()

StickyFOVCircle.Name =
	"StickyFOVCircle"

StickyFOVCircle.Parent =
	FOVGui

local StickyStroke =
	StickyFOVCircle:FindFirstChildOfClass(
		"UIStroke"
	)

if StickyStroke then

	StickyStroke.Color =
		Color3.fromRGB(
			155,
			90,
			255
		)

	StickyStroke.Transparency = 0.65

end

--========================================================
-- TARGET DOT
--========================================================

local TargetDot =
	Instance.new("Frame")

TargetDot.Name =
	"TargetDot"

TargetDot.Size =
	UDim2.fromOffset(
		10,
		10
	)

TargetDot.AnchorPoint =
	Vector2.new(
		0.5,
		0.5
	)

TargetDot.BackgroundColor3 =
	Color3.fromRGB(
		255,
		85,
		100
	)

TargetDot.BorderSizePixel = 0
TargetDot.Visible = false
TargetDot.Parent = FOVGui

createCorner(
	TargetDot,
	50
)

local TargetDotStroke =
	Instance.new("UIStroke")

TargetDotStroke.Color =
	Color3.fromRGB(
		255,
		255,
		255
	)

TargetDotStroke.Thickness = 1
TargetDotStroke.Parent =
	TargetDot

--========================================================
-- TARGET LINE
--========================================================

local TargetLine =
	Instance.new("Frame")

TargetLine.Name =
	"TargetLine"

TargetLine.AnchorPoint =
	Vector2.new(
		0,
		0.5
	)

TargetLine.BackgroundColor3 =
	Color3.fromRGB(
		80,
		220,
		255
	)

TargetLine.BorderSizePixel = 0
TargetLine.Visible = false

TargetLine.Parent =
	FOVGui

local TargetLineCorner =
	Instance.new("UICorner")

TargetLineCorner.CornerRadius =
	UDim.new(
		0,
		10
	)

TargetLineCorner.Parent =
	TargetLine

--========================================================
-- PREDICTION POINT
--========================================================

local PredictionDot =
	Instance.new("Frame")

PredictionDot.Size =
	UDim2.fromOffset(
		8,
		8
	)

PredictionDot.AnchorPoint =
	Vector2.new(
		0.5,
		0.5
	)

PredictionDot.BackgroundColor3 =
	Color3.fromRGB(
		255,
		210,
		80
	)

PredictionDot.BorderSizePixel = 0
PredictionDot.Visible = false

PredictionDot.Parent =
	FOVGui

createCorner(
	PredictionDot,
	50
)

--========================================================
-- TARGET INFO
--========================================================

local TargetInfoGui =
	Instance.new("ScreenGui")

TargetInfoGui.Name =
	"VectorTargetInfo"

TargetInfoGui.ResetOnSpawn =
	false

TargetInfoGui.IgnoreGuiInset =
	true

TargetInfoGui.DisplayOrder =
	999

TargetInfoGui.Parent =
	PlayerGui

local TargetInfo =
	Instance.new("Frame")

TargetInfo.Size =
	UDim2.fromOffset(
		235,
		125
	)

TargetInfo.AnchorPoint =
	Vector2.new(
		0,
		1
	)

TargetInfo.Position =
	UDim2.new(
		1,
		-25,
		1,
		-25
	)

TargetInfo.BackgroundColor3 =
	Color3.fromRGB(
		12,
		16,
		22
	)

TargetInfo.BackgroundTransparency =
	0.08

TargetInfo.BorderSizePixel = 0

TargetInfo.Visible =
	false

TargetInfo.Parent =
	TargetInfoGui

local TargetInfoCorner =
	Instance.new("UICorner")

TargetInfoCorner.CornerRadius =
	UDim.new(
		0,
		12
	)

TargetInfoCorner.Parent =
	TargetInfo

local TargetInfoStroke =
	Instance.new("UIStroke")

TargetInfoStroke.Color =
	Color3.fromRGB(
		70,
		210,
		255
	)

TargetInfoStroke.Thickness = 1
TargetInfoStroke.Transparency = 0.25
TargetInfoStroke.Parent =
	TargetInfo

local TargetInfoText =
	Instance.new("TextLabel")

TargetInfoText.Size =
	UDim2.new(
		1,
		-20,
		1,
		-20
	)

TargetInfoText.Position =
	UDim2.fromOffset(
		10,
		10
	)

TargetInfoText.BackgroundTransparency =
	1

TargetInfoText.TextColor3 =
	Color3.fromRGB(
		220,
		235,
		245
	)

TargetInfoText.TextSize = 11
TargetInfoText.Font =
	Enum.Font.GothamMedium

TargetInfoText.TextXAlignment =
	Enum.TextXAlignment.Left

TargetInfoText.TextYAlignment =
	Enum.TextYAlignment.Top

TargetInfoText.Parent =
	TargetInfo

--========================================================
-- GUI HELPERS
--========================================================

local function createUICorner(parent, radius)

	local corner =
		Instance.new("UICorner")

	corner.CornerRadius =
		UDim.new(
			0,
			radius
		)

	corner.Parent =
		parent

	return corner

end

local function createUIStroke(
	parent,
	color,
	thickness,
	transparency
)

	local stroke =
		Instance.new("UIStroke")

	stroke.Color =
		color

	stroke.Thickness =
		thickness or 1

	stroke.Transparency =
		transparency or 0

	stroke.Parent =
		parent

	return stroke

end

local function createText(
	parent,
	text,
	size,
	font
)

	local label =
		Instance.new("TextLabel")

	label.BackgroundTransparency =
		1

	label.Text =
		text

	label.TextColor3 =
		Color3.fromRGB(
			235,
			240,
			248
		)

	label.TextSize =
		size or 14

	label.Font =
		font
		or Enum.Font.GothamMedium

	label.Parent =
		parent

	return label
end

--========================================================
-- MAIN MENU
--========================================================

local ScreenGui =
	Instance.new("ScreenGui")

ScreenGui.Name =
	"VectorAimESP"

ScreenGui.ResetOnSpawn =
	false

ScreenGui.IgnoreGuiInset =
	true

ScreenGui.DisplayOrder =
	999

ScreenGui.Parent =
	PlayerGui

local Main =
	Instance.new("Frame")

Main.Name =
	"Main"

Main.Size =
	UDim2.fromOffset(
		485,
		600
	)

Main.Position =
	UDim2.new(
		0.5,
		-242,
		0.5,
		-300
	)

Main.BackgroundColor3 =
	Color3.fromRGB(
		12,
		15,
		21
	)

Main.BorderSizePixel = 0

Main.Parent =
	ScreenGui

createUICorner(
	Main,
	17
)

createUIStroke(
	Main,
	Color3.fromRGB(
		65,
		75,
		95
	),
	1.4,
	0.15
)

--========================================================
-- TOP GLOW
--========================================================

local TopGlow =
	Instance.new("Frame")

TopGlow.Size =
	UDim2.new(
		1,
		0,
		0,
		3
	)

TopGlow.BackgroundColor3 =
	Color3.fromRGB(
		60,
		220,
		255
	)

TopGlow.BorderSizePixel = 0

TopGlow.Parent =
	Main

local TopGradient =
	Instance.new("UIGradient")

TopGradient.Color =
	ColorSequence.new({

		ColorSequenceKeypoint.new(
			0,
			Color3.fromRGB(
				60,
				220,
				255
			)
		),

		ColorSequenceKeypoint.new(
			0.5,
			Color3.fromRGB(
				145,
				95,
				255
			)
		),

		ColorSequenceKeypoint.new(
			1,
			Color3.fromRGB(
				60,
				255,
				165
			)
		)

	})

TopGradient.Parent =
	TopGlow

--========================================================
-- HEADER
--========================================================

local Header =
	Instance.new("Frame")

Header.Size =
	UDim2.new(
		1,
		0,
		0,
		82
	)

Header.BackgroundTransparency =
	1

Header.Active =
	true

Header.Parent =
	Main

local Icon =
	Instance.new("Frame")

Icon.Size =
	UDim2.fromOffset(
		47,
		47
	)

Icon.Position =
	UDim2.fromOffset(
		15,
		16
	)

Icon.BackgroundColor3 =
	Color3.fromRGB(
		22,
		30,
		43
	)

Icon.BorderSizePixel =
	0

Icon.Parent =
	Header

createUICorner(
	Icon,
	12
)

createUIStroke(
	Icon,
	Color3.fromRGB(
		70,
		210,
		255
	),
	1,
	0.2
)

local IconLabel =
	createText(
		Icon,
		"◎",
		27,
		Enum.Font.GothamBold
	)

IconLabel.Size =
	UDim2.fromScale(
		1,
		1
	)

IconLabel.TextColor3 =
	Color3.fromRGB(
		90,
		230,
		255
	)

local Title =
	createText(
		Header,
		"VECTOR",
		22,
		Enum.Font.GothamBold
	)

Title.Position =
	UDim2.fromOffset(
		76,
		11
	)

Title.Size =
	UDim2.fromOffset(
		250,
		27
	)

Title.TextXAlignment =
	Enum.TextXAlignment.Left

local Subtitle =
	createText(
		Header,
		"ADVANCED AIM  •  ESP",
		10,
		Enum.Font.GothamBold
	)

Subtitle.Position =
	UDim2.fromOffset(
		77,
		40
	)

Subtitle.Size =
	UDim2.fromOffset(
		250,
		18
	)

Subtitle.TextColor3 =
	Color3.fromRGB(
		105,
		130,
		150
	)

Subtitle.TextXAlignment =
	Enum.TextXAlignment.Left

--========================================================
-- WINDOW BUTTONS
--========================================================

local MinimizeButton =
	Instance.new("TextButton")

MinimizeButton.Size =
	UDim2.fromOffset(
		35,
		35
	)

MinimizeButton.Position =
	UDim2.new(
		1,
		-83,
		0,
		16
	)

MinimizeButton.BackgroundColor3 =
	Color3.fromRGB(
		25,
		31,
		41
	)

MinimizeButton.BorderSizePixel =
	0

MinimizeButton.Text =
	"—"

MinimizeButton.TextColor3 =
	Color3.fromRGB(
		215,
		225,
		238
	)

MinimizeButton.TextSize =
	20

MinimizeButton.Font =
	Enum.Font.GothamBold

MinimizeButton.AutoButtonColor =
	false

MinimizeButton.Parent =
	Header

createUICorner(
	MinimizeButton,
	9
)

local CloseButton =
	Instance.new("TextButton")

CloseButton.Size =
	UDim2.fromOffset(
		35,
		35
	)

CloseButton.Position =
	UDim2.new(
		1,
		-42,
		0,
		16
	)

CloseButton.BackgroundColor3 =
	Color3.fromRGB(
		43,
		26,
		34
	)

CloseButton.BorderSizePixel =
	0

CloseButton.Text =
	"×"

CloseButton.TextColor3 =
	Color3.fromRGB(
		255,
		105,
		128
	)

CloseButton.TextSize =
	25

CloseButton.Font =
	Enum.Font.GothamMedium

CloseButton.AutoButtonColor =
	false

CloseButton.Parent =
	Header

createUICorner(
	CloseButton,
	9
)

--========================================================
-- CONTENT
--========================================================

local Content =
	Instance.new("Frame")

Content.Size =
	UDim2.new(
		1,
		-24,
		1,
		-96
	)

Content.Position =
	UDim2.fromOffset(
		12,
		84
	)

Content.BackgroundTransparency =
	1

Content.Parent =
	Main

--========================================================
-- TABS
--========================================================

local Tabs =
	Instance.new("Frame")

Tabs.Size =
	UDim2.new(
		1,
		0,
		0,
		43
	)

Tabs.BackgroundColor3 =
	Color3.fromRGB(
		19,
		23,
		31
	)

Tabs.BorderSizePixel =
	0

Tabs.Parent =
	Content

createUICorner(
	Tabs,
	10
)

createUIStroke(
	Tabs,
	Color3.fromRGB(
		48,
		59,
		77
	),
	1,
	0.3
)

local AimTab =
	Instance.new("TextButton")

AimTab.Size =
	UDim2.new(
		0.333,
		-4,
		1,
		-6
	)

AimTab.Position =
	UDim2.fromOffset(
		3,
		3
	)

AimTab.BackgroundColor3 =
	Color3.fromRGB(
		37,
		94,
		116
	)

AimTab.BorderSizePixel =
	0

AimTab.Text =
	"AIM"

AimTab.TextColor3 =
	Color3.fromRGB(
		240,
		252,
		255
	)

AimTab.TextSize =
	12

AimTab.Font =
	Enum.Font.GothamBold

AimTab.AutoButtonColor =
	false

AimTab.Parent =
	Tabs

createUICorner(
	AimTab,
	8
)

local ESPTab =
	Instance.new("TextButton")

ESPTab.Size =
	UDim2.new(
		0.333,
		-4,
		1,
		-6
	)

ESPTab.Position =
	UDim2.new(
		0.333,
		1,
		0,
		3
	)

ESPTab.BackgroundColor3 =
	Color3.fromRGB(
		19,
		23,
		31
	)

ESPTab.BorderSizePixel =
	0

ESPTab.Text =
	"ESP"

ESPTab.TextColor3 =
	Color3.fromRGB(
		135,
		150,
		170
	)

ESPTab.TextSize =
	12

ESPTab.Font =
	Enum.Font.GothamBold

ESPTab.AutoButtonColor =
	false

ESPTab.Parent =
	Tabs

createUICorner(
	ESPTab,
	8
)

local DebugTab =
	Instance.new("TextButton")

DebugTab.Size =
	UDim2.new(
		0.333,
		-4,
		1,
		-6
	)

DebugTab.Position =
	UDim2.new(
		0.666,
		1,
		0,
		3
	)

DebugTab.BackgroundColor3 =
	Color3.fromRGB(
		19,
		23,
		31
	)

DebugTab.BorderSizePixel =
	0

DebugTab.Text =
	"DEBUG"

DebugTab.TextColor3 =
	Color3.fromRGB(
		135,
		150,
		170
	)

DebugTab.TextSize =
	12

DebugTab.Font =
	Enum.Font.GothamBold

DebugTab.AutoButtonColor =
	false

DebugTab.Parent =
	Tabs

createUICorner(
	DebugTab,
	8
)

--========================================================
-- PAGES
--========================================================

local function createPage(name)

	local page =
		Instance.new("ScrollingFrame")

	page.Name =
		name

	page.Size =
		UDim2.new(
			1,
			0,
			1,
			-53
		)

	page.Position =
		UDim2.fromOffset(
			0,
			53
		)

	page.BackgroundTransparency =
		1

	page.BorderSizePixel =
		0

	page.ScrollBarThickness =
		4

	page.CanvasSize =
		UDim2.new(
			0,
			0,
			0,
			950
		)

	page.Parent =
		Content

	return page
end

local AimPage =
	createPage(
		"AimPage"
	)

local ESPPage =
	createPage(
		"ESPPage"
	)

local DebugPage =
	createPage(
		"DebugPage"
	)

ESPPage.Visible = false
DebugPage.Visible = false

--========================================================
-- SECTION
--========================================================

local function createSection(
	parent,
	title,
	y,
	height
)

	local section =
		Instance.new("Frame")

	section.Size =
		UDim2.new(
			1,
			0,
			0,
			height
		)

	section.Position =
		UDim2.fromOffset(
			0,
			y
		)

	section.BackgroundColor3 =
		Color3.fromRGB(
			17,
			21,
			28
		)

	section.BorderSizePixel =
		0

	section.Parent =
		parent

	createUICorner(
		section,
		12
	)

	createUIStroke(
		section,
		Color3.fromRGB(
			45,
			56,
			73
		),
		1,
		0.25
	)

	local label =
		createText(
			section,
			title,
			11,
			Enum.Font.GothamBold
		)

	label.Size =
		UDim2.new(
			1,
			-24,
			0,
			25
		)

	label.Position =
		UDim2.fromOffset(
			12,
			5
		)

	label.TextColor3 =
		Color3.fromRGB(
			105,
			215,
			255
		)

	label.TextXAlignment =
		Enum.TextXAlignment.Left

	return section
end

--========================================================
-- TOGGLE
--========================================================

local function createToggle(
	parent,
	text,
	y,
	defaultValue,
	callback
)

	local button =
		Instance.new("TextButton")

	button.Size =
		UDim2.new(
			1,
			-20,
			0,
			40
		)

	button.Position =
		UDim2.fromOffset(
			10,
			y
		)

	button.BackgroundColor3 =
		Color3.fromRGB(
			27,
			32,
			42
		)

	button.BorderSizePixel =
		0

	button.Text =
		""

	button.AutoButtonColor =
		false

	button.Parent =
		parent

	createUICorner(
		button,
		9
	)

	local label =
		createText(
			button,
			text,
			12,
			Enum.Font.GothamMedium
		)

	label.Size =
		UDim2.new(
			1,
			-75,
			1,
			0
		)

	label.Position =
		UDim2.fromOffset(
			12,
			0
		)

	label.TextXAlignment =
		Enum.TextXAlignment.Left

	local state =
		createText(
			button,
			"",
			10,
			Enum.Font.GothamBold
		)

	state.Size =
		UDim2.fromOffset(
			50,
			40
		)

	state.Position =
		UDim2.new(
			1,
			-58,
			0,
			0
		)

	state.TextXAlignment =
		Enum.TextXAlignment.Center

	local value =
		defaultValue

	local function refresh()

		if value then

			button.BackgroundColor3 =
				Color3.fromRGB(
					27,
					64,
					56
				)

			state.Text =
				"ON"

			state.TextColor3 =
				Color3.fromRGB(
					70,
					255,
					170
				)

		else

			button.BackgroundColor3 =
				Color3.fromRGB(
					29,
					32,
					40
				)

			state.Text =
				"OFF"

			state.TextColor3 =
				Color3.fromRGB(
					120,
					130,
					145
				)

		end

	end

	button.MouseButton1Click:Connect(function()

		value =
			not value

		refresh()

		callback(
			value
		)

	end)

	refresh()

	return button
end

--========================================================
-- SLIDER
--========================================================

local function createSlider(
	parent,
	title,
	y,
	minValue,
	maxValue,
	defaultValue,
	callback,
	formatString
)

	local holder =
		Instance.new("Frame")

	holder.Size =
		UDim2.new(
			1,
			-20,
			0,
			55
		)

	holder.Position =
		UDim2.fromOffset(
			10,
			y
		)

	holder.BackgroundTransparency =
		1

	holder.Parent =
		parent

	local label =
		createText(
			holder,
			title,
			11,
			Enum.Font.GothamMedium
		)

	label.Size =
		UDim2.fromOffset(
			190,
			20
		)

	label.Position =
		UDim2.fromOffset(
			0,
			0
		)

	label.TextXAlignment =
		Enum.TextXAlignment.Left

	local valueLabel =
		createText(
			holder,
			"",
			11,
			Enum.Font.GothamBold
		)

	valueLabel.Size =
		UDim2.fromOffset(
			100,
			20
		)

	valueLabel.Position =
		UDim2.new(
			1,
			-100,
			0,
			0
		)

	valueLabel.TextXAlignment =
		Enum.TextXAlignment.Right

	valueLabel.TextColor3 =
		Color3.fromRGB(
			100,
			225,
			255
		)

	local bar =
		Instance.new("Frame")

	bar.Size =
		UDim2.new(
			1,
			0,
			0,
			7
		)

	bar.Position =
		UDim2.fromOffset(
			0,
			29
		)

	bar.BackgroundColor3 =
		Color3.fromRGB(
			33,
			39,
			51
		)

	bar.BorderSizePixel =
		0

	bar.Parent =
		holder

	createUICorner(
		bar,
		5
	)

	local fill =
		Instance.new("Frame")

	fill.BackgroundColor3 =
		Color3.fromRGB(
			70,
			215,
			255
		)

	fill.Size =
		UDim2.fromScale(
			0,
			1
		)

	fill.BorderSizePixel =
		0

	fill.Parent =
		bar

	createUICorner(
		fill,
		5
	)

	local knob =
		Instance.new("Frame")

	knob.Size =
		UDim2.fromOffset(
			13,
			13
		)

	knob.AnchorPoint =
		Vector2.new(
			0.5,
			0.5
		)

	knob.Position =
		UDim2.fromScale(
			0,
			0.5
		)

	knob.BackgroundColor3 =
		Color3.fromRGB(
			230,
			250,
			255
		)

	knob.BorderSizePixel =
		0

	knob.Parent =
		bar

	createUICorner(
		knob,
		20
	)

	local dragging =
		false

	local function displayValue(value)

		if formatString then

			return string.format(
				formatString,
				value
			)

		end

		return tostring(
			math.floor(
				value + 0.5
			)
		)
	end

	local function setValue(value)

		value =
			math.clamp(
				value,
				minValue,
				maxValue
			)

		local percent =
			(
				value
				- minValue
			)
			/
			(
				maxValue
				- minValue
			)

		fill.Size =
			UDim2.fromScale(
				percent,
				1
			)

		knob.Position =
			UDim2.fromScale(
				percent,
				0.5
			)

		valueLabel.Text =
			displayValue(
				value
			)

		callback(
			value
		)
	end

	local function updateFromX(x)

		local relative =
			x
			- bar.AbsolutePosition.X

		local percent =
			math.clamp(
				relative
				/
				bar.AbsoluteSize.X,
				0,
				1
			)

		local value =
			minValue
			+
			(
				maxValue
				- minValue
			)
			* percent

		setValue(
			value
		)
	end

	bar.InputBegan:Connect(function(input)

		if input.UserInputType ==
			Enum.UserInputType.MouseButton1
			or input.UserInputType ==
			Enum.UserInputType.Touch then

			dragging =
				true

			updateFromX(
				input.Position.X
			)

		end

	end)

	UserInputService.InputChanged:Connect(function(input)

		if not dragging then
			return
		end

		if input.UserInputType ==
			Enum.UserInputType.MouseMovement
			or input.UserInputType ==
			Enum.UserInputType.Touch then

			updateFromX(
				input.Position.X
			)

		end

	end)

	UserInputService.InputEnded:Connect(function(input)

		if input.UserInputType ==
			Enum.UserInputType.MouseButton1
			or input.UserInputType ==
			Enum.UserInputType.Touch then

			dragging =
				false

		end

	end)

	setValue(
		defaultValue
	)

	return holder
end

--========================================================
-- CYCLE BUTTON
--========================================================

local function createCycleButton(
	parent,
	text,
	y,
	values,
	defaultIndex,
	callback
)

	local button =
		Instance.new("TextButton")

	button.Size =
		UDim2.new(
			1,
			-20,
			0,
			40
		)

	button.Position =
		UDim2.fromOffset(
			10,
			y
		)

	button.BackgroundColor3 =
		Color3.fromRGB(
			27,
			33,
			43
		)

	button.BorderSizePixel =
		0

	button.Text =
		""

	button.AutoButtonColor =
		false

	button.Parent =
		parent

	createUICorner(
		button,
		9
	)

	local label =
		createText(
			button,
			text,
			12,
			Enum.Font.GothamMedium
		)

	label.Size =
		UDim2.new(
			0.5,
			0,
			1,
			0
		)

	label.Position =
		UDim2.fromOffset(
			12,
			0
		)

	label.TextXAlignment =
		Enum.TextXAlignment.Left

	local valueLabel =
		createText(
			button,
			"",
			11,
			Enum.Font.GothamBold
		)

	valueLabel.Size =
		UDim2.new(
			0.5,
			-12,
			1,
			0
		)

	valueLabel.Position =
		UDim2.new(
			0.5,
			0,
			0,
			0
		)

	valueLabel.TextXAlignment =
		Enum.TextXAlignment.Right

	valueLabel.TextColor3 =
		Color3.fromRGB(
			95,
			220,
			255
		)

	local index =
		defaultIndex

	local function refresh()

		valueLabel.Text =
			tostring(
				values[index]
			)

	end

	button.MouseButton1Click:Connect(function()

		index += 1

		if index >
			#values then

			index = 1

		end

		refresh()

		callback(
			values[index]
		)

	end)

	refresh()

	return button
end

--========================================================
-- AIM PAGE - TARGETING
--========================================================

local TargetSection =
	createSection(
		AimPage,
		"TARGETING",
		0,
		310
	)

createToggle(
	TargetSection,
	"Aim Assist",
	36,
	AIM_ENABLED,
	function(value)

		AIM_ENABLED =
			value

		if not value then
			releaseTarget()
		end

	end
)

createToggle(
	TargetSection,
	"Target Players",
	80,
	TARGET_PLAYERS,
	function(value)

		TARGET_PLAYERS =
			value

	end
)

createToggle(
	TargetSection,
	"Target NPC / Bots",
	124,
	TARGET_NPCS,
	function(value)

		TARGET_NPCS =
			value

	end
)

createToggle(
	TargetSection,
	"Target Lock",
	168,
	TARGET_LOCK,
	function(value)

		TARGET_LOCK =
			value

	end
)

createToggle(
	TargetSection,
	"Sticky Aim",
	212,
	STICKY_AIM,
	function(value)

		STICKY_AIM =
			value

	end
)

createCycleButton(
	TargetSection,
	"Aim Mode",
	256,
	{
		"Hold",
		"Toggle",
		"Always"
	},
	1,
	function(value)

		AIM_MODE =
			value

	end
)

--========================================================
-- AIM PRECISION
--========================================================

local PrecisionSection =
	createSection(
		AimPage,
		"PRECISION",
		322,
		335
	)

createSlider(
	PrecisionSection,
	"FOV Radius",
	35,
	40,
	450,
	FOV_RADIUS,
	function(value)

		FOV_RADIUS =
			math.floor(
				value
			)

	end,
	"%d"
)

createSlider(
	PrecisionSection,
	"Sticky FOV",
	90,
	50,
	500,
	STICKY_FOV,
	function(value)

		STICKY_FOV =
			math.floor(
				value
			)

	end,
	"%d"
)

createSlider(
	PrecisionSection,
	"Max Distance",
	145,
	50,
	2000,
	AIM_DISTANCE,
	function(value)

		AIM_DISTANCE =
			math.floor(
				value
			)

	end,
	"%d"
)

createSlider(
	PrecisionSection,
	"Smoothness",
	200,
	1,
	100,
	SMOOTHNESS * 100,
	function(value)

		SMOOTHNESS =
			value / 100

	end,
	"%d%%"
)

createSlider(
	PrecisionSection,
	"Reaction Delay",
	255,
	0,
	300,
	REACTION_DELAY,
	function(value)

		REACTION_DELAY =
			math.floor(
				value
			)

	end,
	"%d ms"
)

--========================================================
-- AIM OPTIONS
--========================================================

local OptionsSection =
	createSection(
		AimPage,
		"OPTIONS",
		669,
		320
	)

createCycleButton(
	OptionsSection,
	"Target Priority",
	35,
	{
		"Crosshair",
		"Distance",
		"Health",
		"Threat",
		"Custom"
	},
	1,
	function(value)

		TARGET_PRIORITY =
			value

	end
)

createCycleButton(
	OptionsSection,
	"Target Part",
	80,
	{
		"Head",
		"UpperTorso",
		"HumanoidRootPart"
	},
	1,
	function(value)

		TARGET_PART =
			value

	end
)

createCycleButton(
	OptionsSection,
	"Smoothing Mode",
	125,
	{
		"Exponential",
		"Linear",
		"Fast",
		"Slow",
		"Spring"
	},
	1,
	function(value)

		AIM_SMOOTH_MODE =
			value

	end
)

createCycleButton(
	OptionsSection,
	"Wall Check Mode",
	170,
	{
		"Smart",
		"Simple",
		"Strict"
	},
	1,
	function(value)

		WALL_CHECK_MODE =
			value

	end
)

createToggle(
	OptionsSection,
	"Team Check",
	215,
	TEAM_CHECK,
	function(value)

		TEAM_CHECK =
			value

	end
)

createToggle(
	OptionsSection,
	"Ignore Friends",
	259,
	IGNORE_FRIENDS,
	function(value)

		IGNORE_FRIENDS =
			value

	end
)

--========================================================
-- PREDICTION SECTION
--========================================================

local PredictionSection =
	createSection(
		AimPage,
		"PREDICTION",
		1001,
		230
	)

createToggle(
	PredictionSection,
	"Prediction",
	35,
	PREDICTION_ENABLED,
	function(value)

		PREDICTION_ENABLED =
			value

	end
)

createToggle(
	PredictionSection,
	"Auto Prediction",
	79,
	AUTO_PREDICTION,
	function(value)

		AUTO_PREDICTION =
			value

	end
)

createSlider(
	PredictionSection,
	"Prediction Time",
	126,
	0,
	50,
	PREDICTION_TIME * 100,
	function(value)

		PREDICTION_TIME =
			value / 100

	end,
	"%.0f%%"
)

--========================================================
-- ESP PAGE
--========================================================

local ESPControl =
	createSection(
		ESPPage,
		"ESP CONTROL",
		0,
		220
	)

createToggle(
	ESPControl,
	"Master ESP",
	36,
	ESP_ENABLED,
	function(value)

		ESP_ENABLED =
			value

		updateAllESP()

	end
)

createToggle(
	ESPControl,
	"Players",
	81,
	PLAYERS_ENABLED,
	function(value)

		PLAYERS_ENABLED =
			value

		updateAllESP()

	end
)

createToggle(
	ESPControl,
	"NPC / Bots",
	126,
	NPC_ENABLED,
	function(value)

		NPC_ENABLED =
			value

		updateAllESP()

	end
)

createToggle(
	ESPControl,
	"Names",
	171,
	ESP_NAMES,
	function(value)

		ESP_NAMES =
			value

		updateAllESP()

	end
)

local ESPInfo =
	createSection(
		ESPPage,
		"ESP INFORMATION",
		232,
		180
	)

createToggle(
	ESPInfo,
	"Distance",
	36,
	ESP_DISTANCE,
	function(value)

		ESP_DISTANCE =
			value

		updateAllESP()

	end
)

createToggle(
	ESPInfo,
	"Health",
	81,
	ESP_HEALTH,
	function(value)

		ESP_HEALTH =
			value

		updateAllESP()

	end
)

createToggle(
	ESPInfo,
	"Through Walls",
	126,
	true,
	function(value)

		-- Highlight depth remains AlwaysOnTop
		-- intentionally for this local ESP.

	end
)

local ESPVisual =
	createSection(
		ESPPage,
		"AIM VISUALS",
		424,
		275
	)

createToggle(
	ESPVisual,
	"Show FOV",
	36,
	SHOW_FOV,
	function(value)

		SHOW_FOV =
			value

	end
)

createToggle(
	ESPVisual,
	"Target Line",
	81,
	SHOW_TARGET_LINE,
	function(value)

		SHOW_TARGET_LINE =
			value

	end
)

createToggle(
	ESPVisual,
	"Target Dot",
	126,
	SHOW_TARGET_DOT,
	function(value)

		SHOW_TARGET_DOT =
			value

	end
)

createToggle(
	ESPVisual,
	"Target Info",
	171,
	SHOW_TARGET_INFO,
	function(value)

		SHOW_TARGET_INFO =
			value

	end
)

createToggle(
	ESPVisual,
	"Prediction Point",
	216,
	SHOW_PREDICTION_POINT,
	function(value)

		SHOW_PREDICTION_POINT =
			value

	end
)

--========================================================
-- DEBUG PAGE
--========================================================

local DebugStatus =
	createSection(
		DebugPage,
		"LIVE AIM DEBUG",
		0,
		280
	)

local DebugText =
	createText(
		DebugStatus,
		"",
		11,
		Enum.Font.Code
	)

DebugText.Size =
	UDim2.new(
		1,
		-24,
		1,
		-40
	)

DebugText.Position =
	UDim2.fromOffset(
		12,
		36
	)

DebugText.TextColor3 =
	Color3.fromRGB(
		165,
		190,
		205
	)

DebugText.TextXAlignment =
	Enum.TextXAlignment.Left

DebugText.TextYAlignment =
	Enum.TextYAlignment.Top

--========================================================
-- PROFILE SECTION
--========================================================

local ProfilesSection =
	createSection(
		DebugPage,
		"PROFILES",
		292,
		265
	)

local ProfileButton =
	Instance.new("TextButton")

ProfileButton.Size =
	UDim2.new(
		1,
		-20,
		0,
		42
	)

ProfileButton.Position =
	UDim2.fromOffset(
		10,
		35
	)

ProfileButton.BackgroundColor3 =
	Color3.fromRGB(
		29,
		36,
		47
	)

ProfileButton.BorderSizePixel =
	0

ProfileButton.Text =
	"LOAD: LEGIT"

ProfileButton.TextColor3 =
	Color3.fromRGB(
		215,
		235,
		245
	)

ProfileButton.TextSize =
	11

ProfileButton.Font =
	Enum.Font.GothamBold

ProfileButton.AutoButtonColor =
	false

ProfileButton.Parent =
	ProfilesSection

createUICorner(
	ProfileButton,
	9
)

local profileNames = {
	"LEGIT",
	"RAGE",
	"NPC"
}

local profileIndex = 1

local function applyProfile(name)

	local profile =
		Profiles[name]

	if not profile then
		return
	end

	FOV_RADIUS =
		profile.FOV_RADIUS

	STICKY_FOV =
		profile.STICKY_FOV

	AIM_DISTANCE =
		profile.AIM_DISTANCE

	SMOOTHNESS =
		profile.SMOOTHNESS

	PREDICTION_ENABLED =
		profile.PREDICTION_ENABLED

	AUTO_PREDICTION =
		profile.AUTO_PREDICTION

	PREDICTION_TIME =
		profile.PREDICTION_TIME

	TARGET_PRIORITY =
		profile.TARGET_PRIORITY

	TARGET_PART =
		profile.TARGET_PART

	WALL_CHECK =
		profile.WALL_CHECK

	TARGET_LOCK =
		profile.TARGET_LOCK

	STICKY_AIM =
		profile.STICKY_AIM

	releaseTarget()

end

ProfileButton.MouseButton1Click:Connect(function()

	profileIndex += 1

	if profileIndex >
		#profileNames then

		profileIndex = 1

	end

	local name =
		profileNames[
			profileIndex
		]

	ProfileButton.Text =
		"LOAD: "
		.. name

	applyProfile(
		name
	)

end)

local ResetButton =
	Instance.new("TextButton")

ResetButton.Size =
	UDim2.new(
		1,
		-20,
		0,
		42
	)

ResetButton.Position =
	UDim2.fromOffset(
		10,
		87
	)

ResetButton.BackgroundColor3 =
	Color3.fromRGB(
		32,
		34,
		42
	)

ResetButton.BorderSizePixel =
	0

ResetButton.Text =
	"RESET AIM"

ResetButton.TextColor3 =
	Color3.fromRGB(
		220,
		225,
		235
	)

ResetButton.TextSize =
	11

ResetButton.Font =
	Enum.Font.GothamBold

ResetButton.AutoButtonColor =
	false

ResetButton.Parent =
	ProfilesSection

createUICorner(
	ResetButton,
	9
)

ResetButton.MouseButton1Click:Connect(function()

	FOV_RADIUS = 180
	STICKY_FOV = 240
	AIM_DISTANCE = 1000

	SMOOTHNESS = 0.18

	PREDICTION_ENABLED = false
	AUTO_PREDICTION = true
	PREDICTION_TIME = 0.12

	TARGET_PRIORITY = "Crosshair"
	TARGET_PART = "Head"

	WALL_CHECK = false
	TARGET_LOCK = true
	STICKY_AIM = true

	releaseTarget()

end)

local DebugToggle =
	Instance.new("TextButton")

DebugToggle.Size =
	UDim2.new(
		1,
		-20,
		0,
		42
	)

DebugToggle.Position =
	UDim2.fromOffset(
		10,
		139
	)

DebugToggle.BackgroundColor3 =
	Color3.fromRGB(
		29,
		36,
		47
	)

DebugToggle.BorderSizePixel =
	0

DebugToggle.Text =
	"TOGGLE OVERLAY"

DebugToggle.TextColor3 =
	Color3.fromRGB(
		210,
		235,
		245
	)

DebugToggle.TextSize =
	11

DebugToggle.Font =
	Enum.Font.GothamBold

DebugToggle.AutoButtonColor =
	false

DebugToggle.Parent =
	ProfilesSection

createUICorner(
	DebugToggle,
	9
)

DebugToggle.MouseButton1Click:Connect(function()

	debugVisible =
		not debugVisible

end)

--========================================================
-- TAB SWITCH
--========================================================

local function switchTab(name)

	menuPage =
		name

	AimPage.Visible =
		name == "AIM"

	ESPPage.Visible =
		name == "ESP"

	DebugPage.Visible =
		name == "DEBUG"

	AimTab.BackgroundColor3 =
		Color3.fromRGB(
			19,
			23,
			31
		)

	ESPTab.BackgroundColor3 =
		Color3.fromRGB(
			19,
			23,
			31
		)

	DebugTab.BackgroundColor3 =
		Color3.fromRGB(
			19,
			23,
			31
		)

	AimTab.TextColor3 =
		Color3.fromRGB(
			135,
			150,
			170
		)

	ESPTab.TextColor3 =
		Color3.fromRGB(
			135,
			150,
			170
		)

	DebugTab.TextColor3 =
		Color3.fromRGB(
			135,
			150,
			170
		)

	if name == "AIM" then

		AimTab.BackgroundColor3 =
			Color3.fromRGB(
				37,
				94,
				116
			)

		AimTab.TextColor3 =
			Color3.fromRGB(
				240,
				252,
				255
			)

	elseif name == "ESP" then

		ESPTab.BackgroundColor3 =
			Color3.fromRGB(
				34,
				90,
				69
			)

		ESPTab.TextColor3 =
			Color3.fromRGB(
				235,
				255,
				245
			)

	elseif name == "DEBUG" then

		DebugTab.BackgroundColor3 =
			Color3.fromRGB(
				72,
				50,
				104
			)

		DebugTab.TextColor3 =
			Color3.fromRGB(
				250,
				240,
				255
			)

	end

end

AimTab.MouseButton1Click:Connect(function()
	switchTab("AIM")
end)

ESPTab.MouseButton1Click:Connect(function()
	switchTab("ESP")
end)

DebugTab.MouseButton1Click:Connect(function()
	switchTab("DEBUG")
end)

--========================================================
-- AIM KEY
--========================================================

local KeyButton =
	Instance.new("TextButton")

KeyButton.Size =
	UDim2.fromOffset(
		130,
		30
	)

KeyButton.Position =
	UDim2.fromOffset(
		10,
		555
	)

KeyButton.BackgroundColor3 =
	Color3.fromRGB(
		25,
		33,
		43
	)

KeyButton.BorderSizePixel =
	0

KeyButton.Text =
	"KEY: Q"

KeyButton.TextColor3 =
	Color3.fromRGB(
		205,
		230,
		243
	)

KeyButton.TextSize =
	10

KeyButton.Font =
	Enum.Font.GothamBold

KeyButton.AutoButtonColor =
	false

KeyButton.Parent =
	AimPage

createUICorner(
	KeyButton,
	8
)

KeyButton.MouseButton1Click:Connect(function()

	listeningForKey =
		true

	KeyButton.Text =
		"PRESS KEY..."

	KeyButton.TextColor3 =
		Color3.fromRGB(
			85,
			230,
			255
		)

end)

--========================================================
-- DRAGGING
--========================================================

local draggingWindow =
	false

local dragStart =
	nil

local startPosition =
	nil

Header.InputBegan:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		draggingWindow =
			true

		dragStart =
			input.Position

		startPosition =
			Main.Position

	end

end)

Header.InputEnded:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1
		or input.UserInputType ==
		Enum.UserInputType.Touch then

		draggingWindow =
			false

	end

end)

UserInputService.InputChanged:Connect(function(input)

	if not draggingWindow then
		return
	end

	if input.UserInputType ~=
		Enum.UserInputType.MouseMovement
		and input.UserInputType ~=
		Enum.UserInputType.Touch then

		return

	end

	local delta =
		input.Position
		- dragStart

	Main.Position =
		UDim2.new(
			startPosition.X.Scale,
			startPosition.X.Offset
				+ delta.X,

			startPosition.Y.Scale,
			startPosition.Y.Offset
				+ delta.Y
		)

end)

--========================================================
-- MINIMIZE
--========================================================

MinimizeButton.MouseButton1Click:Connect(function()

	menuMinimized =
		not menuMinimized

	if menuMinimized then

		Main.Size =
			UDim2.fromOffset(
				485,
				82
			)

		Content.Visible =
			false

		MinimizeButton.Text =
			"+"

	else

		Main.Size =
			UDim2.fromOffset(
				485,
				600
			)

		Content.Visible =
			true

		MinimizeButton.Text =
			"—"

	end

end)

--========================================================
-- CLOSE
--========================================================

CloseButton.MouseButton1Click:Connect(function()

	menuDestroyed =
		true

	AIM_ENABLED =
		false

	releaseTarget()

	safeDestroy(
		ScreenGui
	)

	safeDestroy(
		FOVGui
	)

	safeDestroy(
		TargetInfoGui
	)

	updateAllESP()

end)

--========================================================
-- KEY INPUT
--========================================================

UserInputService.InputBegan:Connect(function(input, processed)

	-- custom key binding
	if listeningForKey then

		if input.UserInputType ==
			Enum.UserInputType.Keyboard then

			if input.KeyCode ~=
				Enum.KeyCode.Unknown then

				AIM_KEY =
					input.KeyCode

				listeningForKey =
					false

				KeyButton.Text =
					"KEY: "
					.. AIM_KEY.Name

				KeyButton.TextColor3 =
					Color3.fromRGB(
						205,
						230,
						243
					)

			end

		end

		return

	end

	if processed then
		return
	end

	-- ALWAYS mode handled automatically

	if input.KeyCode ==
		AIM_KEY then

		aimHeld =
			true

		if AIM_MODE ==
			"Hold" then

			AIM_ENABLED =
				true

		elseif AIM_MODE ==
			"Toggle" then

			AIM_ENABLED =
				not AIM_ENABLED

			if not AIM_ENABLED then
				releaseTarget()
			end

		end

	end

end)

UserInputService.InputEnded:Connect(function(input)

	if input.KeyCode ==
		AIM_KEY then

		aimHeld =
			false

		if AIM_MODE ==
			"Hold" then

			AIM_ENABLED =
				false

			releaseTarget()

		end

	end

end)

--========================================================
-- CHARACTER SETUP
--========================================================

local function setupPlayer(player)

	player.CharacterAdded:Connect(function(character)

		local humanoid =
			character:WaitForChild(
				"Humanoid",
				5
			)

		task.wait(
			0.15
		)

		updateModelESP(
			character
		)

		if player ==
			LocalPlayer then

			currentTarget =
				nil

		end

	end)

	if player.Character then

		updateModelESP(
			player.Character
		)

	end

end

for _, player in ipairs(
	Players:GetPlayers()
) do

	setupPlayer(
		player
	)

end

Players.PlayerAdded:Connect(
	setupPlayer
)

--========================================================
-- NEW OBJECTS
--========================================================

Workspace.DescendantAdded:Connect(function(object)

	if menuDestroyed then
		return
	end

	if object:IsA("Model") then

		task.wait(
			0.10
		)

		if object:FindFirstChildOfClass(
			"Humanoid"
		) then

			updateModelESP(
				object
			)

		end

	end

end)

--========================================================
-- AIM LOOP
--========================================================

local accumulatedScan =
	0

local accumulatedCandidateRefresh =
	0

local function updateFOVVisuals()

	if menuDestroyed then
		return
	end

	local mouse =
		UserInputService:GetMouseLocation()

	FOVCircle.Position =
		UDim2.fromOffset(
			mouse.X,
			mouse.Y
		)

	FOVCircle.Size =
		UDim2.fromOffset(
			FOV_RADIUS * 2,
			FOV_RADIUS * 2
		)

	FOVCircle.Visible =
		SHOW_FOV

		and
		AIM_ENABLED

	StickyFOVCircle.Position =
		UDim2.fromOffset(
			mouse.X,
			mouse.Y
		)

	StickyFOVCircle.Size =
		UDim2.fromOffset(
			STICKY_FOV * 2,
			STICKY_FOV * 2
		)

	StickyFOVCircle.Visible =
		STICKY_AIM
		and AIM_ENABLED
		and currentTarget ~= nil

end

local function updateTargetVisuals()

	if not currentTarget
		or not currentTarget.Parent then

		TargetDot.Visible =
			false

		TargetLine.Visible =
			false

		PredictionDot.Visible =
			false

		return

	end

	local position =
		currentTarget.Position

	local screenPosition,
		visible =
		Camera:WorldToViewportPoint(
			position
		)

	if not visible
		or screenPosition.Z <= 0 then

		TargetDot.Visible =
			false

		TargetLine.Visible =
			false

		PredictionDot.Visible =
			false

		return

	end

	TargetDot.Visible =
		SHOW_TARGET_DOT
		and AIM_ENABLED

	TargetDot.Position =
		UDim2.fromOffset(
			screenPosition.X,
			screenPosition.Y
		)

	-- target line
	if SHOW_TARGET_LINE
		and AIM_ENABLED then

		local mouse =
			UserInputService:GetMouseLocation()

		local target =
			Vector2.new(
				screenPosition.X,
				screenPosition.Y
			)

		local delta =
			target
			-
			mouse

		local length =
			delta.Magnitude

		if length > 2 then

			TargetLine.Visible =
				true

			TargetLine.Position =
				UDim2.fromOffset(
					mouse.X,
					mouse.Y
				)

			TargetLine.Size =
				UDim2.fromOffset(
					length,
					2
				)

			TargetLine.Rotation =
				math.deg(
					math.atan2(
						delta.Y,
						delta.X
					)
				)

		else

			TargetLine.Visible =
				false

		end

	else

		TargetLine.Visible =
			false

	end

	-- Prediction point
	if PREDICTION_ENABLED
		and SHOW_PREDICTION_POINT
		and AIM_ENABLED then

		local predicted =
			getPredictedPosition(
				currentTarget
			)

		local predictedScreen,
			predictedVisible =
			Camera:WorldToViewportPoint(
				predicted
			)

		if predictedVisible
			and predictedScreen.Z > 0 then

			PredictionDot.Visible =
				true

			PredictionDot.Position =
				UDim2.fromOffset(
					predictedScreen.X,
					predictedScreen.Y
				)

		else

			PredictionDot.Visible =
				false

		end

	else

		PredictionDot.Visible =
			false

	end

end

local function updateTargetInfo()

	if not SHOW_TARGET_INFO
		or not AIM_ENABLED
		or not currentTarget
		or not currentTarget.Parent then

		TargetInfo.Visible =
			false

		return

	end

	local character =
		currentTarget.Parent

	local humanoid =
		getCharacterHumanoid(
			character
		)

	local player =
		getPlayerFromCharacter(
			character
		)

	local distance =
		getDistanceToTarget(
			currentTarget
		)

	local screenDistance =
		getScreenDistance(
			currentTarget
		)

	local velocity =
		getVelocity(
			currentTarget
		)

	local targetName =
		player
		and player.DisplayName
		or "NPC / BOT"

	local hp = 0
	local maxHp = 0

	if humanoid then

		hp =
			math.floor(
				humanoid.Health
			)

		maxHp =
			math.floor(
				humanoid.MaxHealth
			)

	end

	local predictionTime =
		calculatePredictionTime(
			distance
		)

	TargetInfoText.Text =
		"TARGET\n"
		..
		targetName
		..
		"\n\n"
		..
		"Distance: "
		..
		math.floor(
			distance
		)
		..
		"\n"
		..
		"HP: "
		..
		hp
		..
		" / "
		..
		maxHp
		..
		"\n"
		..
		"Screen: "
		..
		math.floor(
			screenDistance
		)
		..
		" px\n"
		..
		"Velocity: "
		..
		math.floor(
			velocity.Magnitude
		)
		..
		"\n"
		..
		"Prediction: "
		..
		string.format(
			"%.2f",
			predictionTime
		)

	TargetInfo.Visible =
		true

end

local function updateDebug()

	if not debugVisible then

		DebugText.Text =
			"Debug overlay is disabled.\n\n"
			..
			"Press TOGGLE OVERLAY in the DEBUG tab."

		return

	end

	local candidateCount =
		#targetCandidates

	local currentName =
		"NONE"

	if currentTarget
		and currentTarget.Parent then

		local player =
			getPlayerFromCharacter(
				currentTarget.Parent
			)

		currentName =
			player
			and player.DisplayName
			or "NPC / BOT"

	end

	local currentDistance =
		0

	local currentScreen =
		0

	local currentHealth =
		0

	local currentMaxHealth =
		0

	if currentTarget
		and currentTarget.Parent then

		currentDistance =
			getDistanceToTarget(
				currentTarget
			)

		currentScreen =
			getScreenDistance(
				currentTarget
			)

		local humanoid =
			getCharacterHumanoid(
				currentTarget.Parent
			)

		if humanoid then

			currentHealth =
				humanoid.Health

			currentMaxHealth =
				humanoid.MaxHealth

		end

	end

	local predictionTime =
		currentTarget
		and calculatePredictionTime(
			currentDistance
		)
		or 0

	DebugText.Text =
		"VECTOR AIM DEBUG"
		..
		"\n"
		..
		"────────────────────────"
		..
		"\n"
		..
		"AIM: "
		..
		(
			AIM_ENABLED
			and "ACTIVE"
			or "OFF"
		)
		..
		"\n"
		..
		"Mode: "
		..
		AIM_MODE
		..
		"\n"
		..
		"Priority: "
		..
		TARGET_PRIORITY
		..
		"\n"
		..
		"Target Part: "
		..
		TARGET_PART
		..
		"\n\n"
		..
		"Candidates: "
		..
		candidateCount
		..
		"\n"
		..
		"Target: "
		..
		currentName
		..
		"\n"
		..
		"Distance: "
		..
		string.format(
			"%.1f",
			currentDistance
		)
		..
		"\n"
		..
		"Screen Distance: "
		..
		math.floor(
			currentScreen
		)
		..
		" px"
		..
		"\n"
		..
		"Health: "
		..
		math.floor(
			currentHealth
		)
		..
		" / "
		..
		math.floor(
			currentMaxHealth
		)
		..
		"\n\n"
		..
		"FOV: "
		..
		FOV_RADIUS
		..
		"\n"
		..
		"Sticky FOV: "
		..
		STICKY_FOV
		..
		"\n"
		..
		"Distance Limit: "
		..
		AIM_DISTANCE
		..
		"\n"
		..
		"Smooth: "
		..
		math.floor(
			SMOOTHNESS * 100
		)
		..
		"%"
		..
		"\n"
		..
		"Prediction: "
		..
		(
			PREDICTION_ENABLED
			and "ON"
			or "OFF"
		)
		..
		"\n"
		..
		"Prediction Time: "
		..
		string.format(
			"%.2f",
			predictionTime
		)

end

--========================================================
-- MAIN RENDER LOOP
--========================================================

RunService.RenderStepped:Connect(function(dt)

	if menuDestroyed then
		return
	end

	-- ALWAYS mode
	if AIM_MODE ==
		"Always" then

		AIM_ENABLED =
			true

	end

	-- candidate refresh
	accumulatedCandidateRefresh +=
		dt

	if accumulatedCandidateRefresh >=
		candidateScanInterval then

		accumulatedCandidateRefresh =
			0

		scanTargets()

	end

	updateFOVVisuals()

	-- aim system
	if AIM_ENABLED then

		if not currentTarget then

			acquireTarget()

		else

			if not isCurrentTargetValid() then

				releaseTarget()

				acquireTarget()

			end

		end

		if currentTarget then

			local reactionElapsed =
				(
					os.clock()
					-
					targetAcquiredTime
				)
				* 1000

			if reactionElapsed >=
				REACTION_DELAY then

				aimAtTarget(
					currentTarget,
					dt
				)

			end

		end

	else

		if AIM_MODE ~=
			"Hold" then

			if not aimHeld then
				releaseTarget()
			end

		end

	end

	updateTargetVisuals()
	updateTargetInfo()

	-- debug update
	accumulatedScan +=
		dt

	if accumulatedScan >=
		0.08 then

		accumulatedScan =
			0

		updateDebug()

	end

end)

--========================================================
-- INITIAL START
--========================================================

task.wait(
	0.5
)

scanTargets()

updateAllESP()

switchTab(
	"AIM"
)
