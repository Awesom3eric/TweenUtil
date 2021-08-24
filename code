local TweenUtil = {}

local TweenService = game:GetService("TweenService")

local function create(instance, parent, name, properties)
	local object = Instance.new(instance)
	if properties then
		for propertyName, propertyValue in pairs(properties) do
			object[propertyName] = propertyValue
		end
	end
	if name then object.Name = name end
	if parent then object.Parent = parent end
	return object
end

local weldedModels = {}
function TweenUtil.unweldModel(model)
	for _, part in ipairs(model:GetDescendants()) do
		if part:IsA("BasePart") then
			part.Anchored = true
			if part:FindFirstChild("TweenWeld") then
				part.TweenWeld:Destroy()
			end
		end
	end
end

function TweenUtil.weldModel(model)	
	if model:IsA("Model") then
		TweenUtil.unweldModel(model)
		
		local center = model:FindFirstChild("Center")
		if not center then
			local cframe, size = model:GetBoundingBox()
			center = create("Part", model, "Center", {
				Anchored = true, CanCollide = false, Size = Vector3.new(0, 0, 0), Transparency = 1, CFrame = cframe
			})
		end
		
		local primaryPart = model.PrimaryPart or center
		primaryPart.Anchored = true
		
		for _, part in ipairs(model:GetDescendants()) do
			if part ~= primaryPart then
				local weld = create("WeldConstraint", part, "TweenWeld", {Part0 = part, Part1 = primaryPart})
				part.Anchored = false
			end
		end
		
		weldedModels[model] = true
		return model
	end
end

local tweens = {}
local styles = {"Back", "Bounce", "Circular", "Cubic", "Elastic", "Exponential", "Linear", "Quad", "Quart", "Quint", "Sine"}
local directions = {"In", "Out", "InOut"}
local functions = {"TweenFromCenter", "TweenFromPrimaryPart", "TweenFromPart", "Play", "Shift"}

function TweenUtil.cancelTween(model)
	if tweens[model] then
		tweens[model]:Cancel()
		tweens[model] = nil
	end
end


function TweenUtil.tween(model, ...)
	TweenUtil.cancelTween(model)
	if not weldedModels[model] then
		TweenUtil.weldModel(model)
	end
	for _, part in ipairs(model:GetDescendants()) do
		if not part:FindFirstChildOfClass("WeldConstraint") then
			TweenUtil.weldModel(model)
			break
		end
	end
	
	local args = {...}
	local data = {
		Duration 		= 1,
		EasingStyle 	= Enum.EasingStyle.Quad,
		EasingDirection = Enum.EasingDirection.InOut,
		Repeat 			= 0,
		Reverse			= false,
		Delay 			= 0,
		TweenInfo		= nil,

		Target 			= nil,
		Origin 			= nil,
		Functions		= {"TweenFromPrimaryPart"},
		CustomPart		= nil,
	}

	for _, arg in ipairs(args) do
		local stringArg = tostring(arg)
		if typeof(arg) == "EnumItem" then
			if string.find(stringArg, "Enum.EasingStyle.") then
				data.EasingStyle = arg
			elseif string.find(stringArg, "Enum.EasingDirection.") then
				data.EasingDirection = arg
			end
		elseif typeof(arg) == "TweenInfo" then
			data.TweenInfo = arg
		elseif typeof(arg) == "number" then
			data.Duration = arg
		elseif typeof(arg) == "string" then
			if table.find(styles, arg) then
				data.EasingStyle = Enum.EasingStyle[arg]
			elseif table.find(directions, arg) then
				data.EasingDirection = Enum.EasingDirection[arg]
			elseif arg == "Reverse" then
				data.Reverse = true
			elseif string.find(arg, "Delay ") then
				local delayTime = tonumber(string.sub(arg, #"Delay " + 1, #arg))
				if delayTime then
					data.Delay = delayTime
				else
					warn('Delay is not formatted correctly. Format example: "Delay 5"')
					return
				end
			elseif string.find(arg, "Repeat ") then
				local repeatCount = tonumber(string.sub(arg, #"Repeat " + 1, #arg))
				if repeatCount then
					data.Repeat = repeatCount
				else
					warn('Repeat is not formatted correctly. Format example "Repeat 3"')
				end
			elseif table.find(functions, arg) then
				table.insert(data.Functions, arg)
			end
		elseif typeof(arg) == "CFrame" then
			data.Target = arg
		elseif typeof(arg) == "Vector3" then
			data.Target = CFrame.new(arg)
		elseif typeof(arg) == "Instance" then
			data.CustomPart = arg
		end
	end

	if table.find(data.Functions, "TweenFromCenter") then
		data.Origin = model.Center
	elseif table.find(data.Functions, "TweenFromPart") then
		if data.CustomPart == nil  then
			warn("TweenFromPart part is not identified.")
			return
		else
			if data.CustomPart:IsDescendantOf(model) then
				data.Origin = data.CustomPart
			else
				warn("TweenFromPart part is not parented to model")
				return
			end
		end
	elseif table.find(data.Functions, "TweenFromPrimaryPart") then
		if model.PrimaryPart then
			data.Origin = model.PrimaryPart
		else
			warn("TweenFromPrimaryPart model does not have a PrimaryPart")
			return
		end
	end

	if not data.TweenInfo then
		data.TweenInfo = TweenInfo.new(data.Duration, data.EasingStyle, data.EasingDirection, data.Repeat, data.Reverse, data.Delay)
	end
	
	if table.find(data.Functions, "Shift") then
		data.Target = data.Origin.CFrame * data.Target
	end
	
	local tween = TweenService:Create(data.Origin, data.TweenInfo, {CFrame = data.Target})
	if table.find(data.Functions, "Play") then
		tween:Play()
	end
	return tween
end

return TweenUtil
