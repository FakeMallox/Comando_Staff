# resource_manifest_version "44febabe-d386-4d18-afbe-5e627f4af937"

client_script "staff-c.lua"

local group 

RegisterNetEvent('staff:setGroup')
AddEventHandler('staff:setGroup', function(g)
	print('group setted ' .. g)
	group = g
end)

RegisterNetEvent("textsent")
AddEventHandler('textsent', function(tPID, names2)
	TriggerEvent('chatMessage', "", {205, 205, 0}, " Staff sent to:^0 " .. names2 .."  ".."^0  - " .. tPID)
end)

RegisterNetEvent("textmsg")
AddEventHandler('textmsg', function(source, textmsg, names2, names3 )
	TriggerEvent('chatMessage', "", {205, 205, 0}, "  MOD " .. names3 .."  ".."^0: " .. textmsg)
end)

RegisterNetEvent('sendStaff')
AddEventHandler('sendStaff', function(id, name, message)
  local myId = PlayerId()
  local pid = GetPlayerFromServerId(id)
  if pid == myId then
    TriggerEvent('chatMessage', "", {255, 0, 0}, "STAFF Ci sono Admin online!")
	  TriggerEvent('chatMessage', "", {255, 0, 0}, " [STAFF] | [".. id .."]" .. name .."  "..":^0  " .. message)
  elseif group ~= 'user' and pid ~= myId then
    TriggerEvent('chatMessage', "", {255, 0, 0}, " [STAFF] | [".. id .."]" .. name .."  "..":^0  " .. message)
  end
end)

RegisterCommand("staff", function()
    command = stringsplit(message, " ")
    msg("Uno staffer sarà da te in pochi minuti attendi grazie!")
end, false)

function msg(text)
    TriggerEvent("chatMessage", "[SERVER]", {255,0,0}, text)
end)


function stringsplit(inputstr, sep)
    if sep == nil then
        sep = "%s"
    end
    local t={} ; i=1
    for str in string.gmatch(inputstr, "([^"..sep.."]+)") do
        t[i] = str
        i = i + 1
    end
    return t
    end
end)

function getIdentity(source)
	local identifier = GetPlayerIdentifiers(source)[1]
	local result = MySQL.Sync.fetchAll("SELECT * FROM users WHERE identifier = @identifier", {['@identifier'] = identifier})
	if result[1] ~= nil then
		local identity = result[1]

		return {
			identifier = identity['identifier'],
			name = identity['name'],
			firstname = identity['firstname'],
			lastname = identity['lastname'],
			dateofbirth = identity['dateofbirth'],
			sex = identity['sex'],
			height = identity['height'],
			job = identity['job'],
			group = identity['group']
		}
	else
		return nil
	end
end


function loadExistingPlayers()
	TriggerEvent("es:getPlayers", function(curPlayers)
		for k,v in pairs(curPlayers)do
			TriggerClientEvent("staff:setGroup", v.get('source'), v.get('group'))
		end
	end)
end

loadExistingPlayers()

AddEventHandler('es:playerLoaded', function(Source, user)
	TriggerClientEvent('staff:setGroup', Source, user.getGroup())
end)

AddEventHandler('chatMessage', function(source, color, msg)
	cm = stringsplit(msg, " ")
	if cm[1] == "/staff" or cm[1] == "/staff" then
		CancelEvent()
		if tablelength(cm) > 1 then
			local tPID = tonumber(cm[2])
			local names2 = GetPlayerName(tPID)
			local names3 = GetPlayerName(source)
			local textmsg = ""
			for i=1, #cm do
				if i ~= 1 and i ~=2 then
					textmsg = (textmsg .. " " .. tostring(cm[i]))
				end
			end
			local grupos = getIdentity(source)
		    if grupos.group ~= 'user' then
			    TriggerClientEvent('textmsg', tPID, source, textmsg, names2, names3)
			    TriggerClientEvent('textsent', source, tPID, names2)
		    else
			    TriggerClientEvent('chatMessage', source, "SERVER", {255, 0, 0}, "Permessi Insufficenti!")
			end
		end
	end	

function tablelength(T)
	local count = 0
	for _ in pairs(T) do count = count + 1 end
	return count
end
