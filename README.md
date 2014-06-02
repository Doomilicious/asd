asd
===
            /**
             * This is the file where the bot commands are located
             *
             * @license MIT license
             */
             
            var http = require('http');
            var sys = require('sys');
             
            exports.commands = {
                            set: function(arg, by, room, con) {
                                    if (!this.hasRank(by, '%@&#~') || room.charAt(0) === ',') return false;
     
                                    var settable = {
                                            ce: 1,
                                            menu: 1,
                                            facepalm: 1,
                                            stabmons: 1,
                                            clap: 1,
                                            guida: 1,
                                            kermit: 1,
                                            hehe: 1,
                                            crathuman: 1,
                                            stop: 1

                                    };
                                    var modOpts = {
                                            flooding: 1,
                                            caps: 1,
                                            stretching: 1,
                                            bannedwords: 1,
                                            snen: 1
                                    };
     
                                    var opts = arg.split(',');
                                    var cmd = toId(opts[0]);
                                    if (cmd === 'mod' || cmd === 'm' || cmd === 'modding') {
                                            if (!opts[1] || !toId(opts[1]) || !(toId(opts[1]) in modOpts)) return this.say(con, room, 'Incorrect command: correct syntax is .set mod, [' +
                                                    Object.keys(modOpts).join('/') + '](, [on/off])');
     
                                            if (!this.settings['modding']) this.settings['modding'] = {};
                                            if (!this.settings['modding'][room]) this.settings['modding'][room] = {};
                                            if (opts[2] && toId(opts[2])) {
                                                    if (!this.hasRank(by, '#~')) return false;
                                                    if (!(toId(opts[2]) in {on: 1, off: 1}))  return this.say(con, room, 'Incorrect command: correct syntax is .set mod, [' +
                                                            Object.keys(modOpts).join('/') + '](, [on/off])');
                                                    if (toId(opts[2]) === 'off') {
                                                            this.settings['modding'][room][toId(opts[1])] = 0;
                                                    } else {
                                                            delete this.settings['modding'][room][toId(opts[1])];
                                                    }
                                                    this.writeSettings();
                                                    this.say(con, room, 'Moderation for ' + toId(opts[1]) + ' in this room is now ' + toId(opts[2]).toUpperCase() + '.');
                                                    return;
                                            } else {
                                                    this.say(con, room, 'Moderation for ' + toId(opts[1]) + ' in this room is currently ' +
                                                            (this.settings['modding'][room][toId(opts[1])] === 0 ? 'OFF' : 'ON') + '.');
                                                    return;
                                            }
                                    } else {
                                            if (!Commands[cmd]) return this.say(con, room, '.' + opts[0] + ' is not a valid command.');
                                            var failsafe = 0;
                                            while (!(cmd in settable)) {
                                                    if (typeof Commands[cmd] === 'string') {
                                                            cmd = Commands[cmd];
                                                    } else if (typeof Commands[cmd] === 'function') {
                                                            if (cmd in settable) {
                                                                    break;
                                                            } else {
                                                                    this.say(con, room, 'The settings for .' + opts[0] + ' cannot be changed.');
                                                                    return;
                                                            }
                                                    } else {
                                                            this.say(con, room, 'Something went wrong. PM TalkTakesTime here or on Smogon with the command you tried.');
                                                            return;
                                                    }
                                                    failsafe++;
                                                    if (failsafe > 5) {
                                                            this.say(con, room, 'The command ".' + opts[0] + '" could not be found.');
                                                            return;
                                                    }
                                            }
     
                                            var settingsLevels = {
                                                    off: false,
                                                    disable: false,
                                                    '+': '+',
                                                    '%': '%',
                                                    '@': '@',
                                                    '&': '&',
                                                    '#': '#',
                                                    '~': '~',
                                                    on: true,
                                                    enable: true
                                            };
                                            if (!opts[1] || !opts[1].trim()) {
                                                    var msg = '';
                                                    if (!this.settings[cmd] || (!this.settings[cmd][room] && this.settings[cmd][room] !== false)) {
                                                            msg = '.' + cmd + ' is available for users of rank ' + (cmd === 'autoban' ? '#' : config.defaultrank) + ' and above.';
                                                    } else if (this.settings[cmd][room] in settingsLevels) {
                                                            msg = '.' + cmd + ' is available for users of rank ' + this.settings[cmd][room] + ' and above.';
                                                    } else if (this.settings[cmd][room] === true) {
                                                            msg = '.' + cmd + ' is available for all users in this room.';
                                                    } else if (this.settings[cmd][room] === false) {
                                                            msg = '.' + cmd + ' is not available for use in this room.';
                                                    }
                                                    this.say(con, room, msg);
                                                    return;
                                            } else {
                                                    if (!this.hasRank(by, '#~')) return false;
                                                    var newRank = opts[1].trim();
                                                    if (!(newRank in settingsLevels)) return this.say(con, room, 'Unknown option: "' + newRank + '". Valid settings are: off/disable, +, %, @, &, #, ~, on/enable.');
                                                    if (!this.settings[cmd]) this.settings[cmd] = {};
                                                    this.settings[cmd][room] = settingsLevels[newRank];
                                                    this.writeSettings();
                                                    this.say(con, room, 'The command .' + cmd + ' is now ' +
                                                            (settingsLevels[newRank] === newRank ? ' available for users of rank ' + newRank + ' and above.' :
                                                            (this.settings[cmd][room] ? 'available for all users in this room.' : 'unavailable for use in this room.')))
                                            }
                                    }
                            },
                   
                                    reload: function(arg, by, room, con) {
                                            if (!this.hasRank(by, '#~')) return false;
                                            try {
                                                    this.uncacheTree('./commands.js');
                                                    Commands = require('./commands.js').commands;
                                                    this.say(con, room, 'Commands reloaded.');
                                            } catch (e) {
                                                    error('failed to reload: ' + sys.inspect(e));
                                            }
                                    },
                    Menu: 'menu', links: 'menu', utilitylinks: 'menu',
                    menu: function(arg, by, room, con) {
                            if (!this.canUse('informations', room, by) || room.charAt(0) === ',') return false
                            var text = '/declare <p align="center"><font size=3>Utility Links</font><br><br><a href="http://pokemonshowdown.com/damagecalc/"><img src="http://antonlife001.altervista.org/pkmn_skin/bott1hover.png"></a><br><a href="http://pokemondb.net/tools/type-coverage"><img src="http://antonlife001.altervista.org/pkmn_skin/bott2hover.png"></a><br><a href="http://www.serebii.net/games/iv-calcxy.shtml"><img src="http://antonlife001.altervista.org/pkmn_skin/bott3hover.png"></a></p>';
                            this.say(con, room, text);
                    },

                    CE: 'ce',     
                    ce: function(arg, by, room, con) {
                            if (!this.canUse('ce', room, by) || room.charAt(0) === ',') return false
                            var text = '/declare <div class="infobox" target="_blank"><center><font size=3>Catch & Evolve Tournaments</font></center><table border="0" target="_blank"><tbody><tr target="_blank"><td target="_blank">Hosted by:<u> <i>Marvel Six</i></u><br target="_blank">Click here to join the room: <button name="joinRoom" value="ce" target="_blank">C&E</button> </td><td target="_blank"> </td><td target="_blank"> </td><td target="_blank"> </td><td target="_blank"><img src="http://media.pokemoncentral.it/wiki/thumb/5/56/Artwork656.png/200px-Artwork656.png" alt="" width="40" height="40" target="_blank"></td></tr></tbody></table></div>';
                            this.say(con, room, text);
                    },

                    Facepalm: 'facepalm', fp: 'facepalm', palmface: 'facepalm',     
                    facepalm: function(arg, by, room, con) {
                            if (!this.canUse('facepalm', room, by) || room.charAt(0) === ',') return false
                            var text = '/img http://www.blogcdn.com/massively.joystiq.com/media/2011/10/facepalm.jpg, 30';
                            this.say(con, room, text);
                    },

                    Stabmons: 'stabmons', STABmons: 'stabmons', stabmon: 'stabmons',     
                    stabmons: function(arg, by, room, con) {
                            if (!this.canUse('stabmons', room, by) || room.charAt(0) === ',') return false
                            var text = '/declaregreen <ul><li><a href="http://www.smogon.com/forums/threads/stabmons-see-post-2-for-ban-considerations.3493081/">Click here to read the STABmons post on Smogon</a></li><ul>';
                            this.say(con, room, text);

                    },

                    clapclap: 'clap', joker: 'clap', Clap: 'clap',     
                    clap: function(arg, by, room, con) {
                            if (!this.canUse('clap', room, by) || room.charAt(0) === ',') return false
                            var text = '/img http://img1.wikia.nocookie.net/__cb20081127235516/uncyclopedia/images/9/95/Joker_clap.gif, 25';
                            this.say(con, room, text);
                    },

                    comandi: 'guida', help: 'guida', Guida: 'guida',     
                    guida: function(arg, by, room, con) {
                            if (!this.canUse('guida', room, by) || room.charAt(0) === ',') return false
                            var text = '/declarered <ul><li><a href="http://pokemonglobal.forumcommunity.net/?t=56391198">Click here to read the Guide post on our forum</a></li><ul>';
                            this.say(con, room, text);
                    },

                    fap: 'kermit',    
                    kermit: function(arg, by, room, con) {
                            if (!this.canUse('kermit', room, by) || room.charAt(0) === ',') return false
                            var text = '/img http://i.imgur.com/YJkwb.gif, 30';
                            this.say(con, room, text);
                    },
   
                    stop: function(arg, by, room, con) {
                            if (!this.canUse('stop', room, by) || room.charAt(0) === ',') return false
                            var text = '/declarered <center><font size=5>STOP SPAMMING!</font></center>';
                            this.say(con, room, text);
                    },

                    hehe: function(arg, by, room, con) {
                            if (!this.canUse('hehe', room, by) || room.charAt(0) === ',') return false
                            var text = '/img http://x4.fjcdn.com/comments/3769586+_1d5fe9b907bdfeb8aa113bd35cca705f.png, 30';
                            this.say(con, room, text);
                    },

                    crathuman: function(arg, by, room, con) {
                            if (!this.canUse('crathuman', room, by) || room.charAt(0) === ',') return false
                            var text = 'Stealing old men from the streets and giving money to poor people is what I do. I am CratHuman!';
                            this.say(con, room, text);
                    },

                    tell: 'say',
                    say: function(arg, by, room, con) {
                            if (!this.canUse('say', room, by)) return false;
                            this.say(con, room, stripCommands(arg) + ' (' + by + ' said this)');
                    },
            },
