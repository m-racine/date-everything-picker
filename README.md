Graphics courtesy of https://fireemblem.fandom.com



# Favorite picker

Broadly, this is a tool that allows users to pick certain items from a predefined set over others, repeatedly, until a single favorite is found, and then continue to pick for second place and so on, constructing a list. It is a generalization of the [Favorite Pokémon Picker](https://www.dragonflycave.com/favorite.html). If you haven't seen that tool before, look there and play around with it to get a better idea of what this is about.

##### Filtering

This is one of the most obvious use cases for settings: filter the items down to a smaller subset according to the user's selections.

In most cases, you can do this by setting the `shouldIncludeItem` function in the `Picker` options. This is a function that takes two arguments, `item` (one of the item objects you defined) and `settings`, and should return `true` (or another truthy value) if this item should be included given these settings. In order to make such settings work, you may want to add custom properties to your items - you can add any custom properties you like! For example:

```
<p>Include roles:
<label><input type="checkbox" name="roles" class="roles" value="lawyer"> Lawyer</label>
<label><input type="checkbox" name="roles" class="roles" value="assistant"> Assistant</label>
<label><input type="checkbox" name="roles" class="roles" value="defendant"> Defendant</label>
<label><input type="checkbox" name="roles" class="roles" value="witness"> Witness</label>
<label><input type="checkbox" name="roles" class="roles" value="other"> Other</label>
</p>

<p><label>Recurring characters only? <input type="checkbox" id="recurring-only"></label></p>

...

var items = [
    {id: 'phoenix', name: 'Phoenix Wright', image: 'phoenixwright.png', role: 'lawyer', recurring: true},
    {id: 'edgeworth', name: 'Miles Edgeworth', image: 'edgeworth.png', role: 'lawyer', recurring: true},
    {id: 'maya', name: 'Maya Fey', image: 'mayafey.png', role: 'assistant', recurring: true},
    {id: 'redd-white', name: 'Redd White', image: 'reddwhite.png', role: 'witness', recurring: false}
    ...
];

var myPicker = new picker.Picker({
    items: items,
    localStorageKey: 'picker-state',
    defaultSettings: {
        roles: ['lawyer', 'assistant', 'defendant', 'witness', 'other'],
        recurringOnly: true
    },
    shouldIncludeItem: function(item, settings) {
        // Include only if:
        // 1. the character's role is in the list of roles we've checked, and
        // 2. we haven't checked the "recurring only" box, or the character is a recurring character.
        return settings.roles.indexOf(item.role) !== -1 && (!settings.recurringOnly || item.recurring);
    }
});

var pickerUI = new PickerUI(myPicker, {
    elements: {
        pick: "#pick",
        pass: "#pass",
        undo: "#undo",
        redo: "#redo",
        evaluating: "#evaluating",
        favorites: "#favorites",
        settings: {
            roles: '.roles',
            recurringOnly: '#recurring-only'
        }
    }
});
```

Sometimes, though, you may need more complex filtering than just independently determining inclusion for each item on its own. For these cases, you can use the `getFilteredItems` option, which is a function that takes the current settings and should simply return a list of item identifiers that should be included given these settings. Note that specifying this function will completely override `shouldIncludeItem`! For example:

```
var items = [
    {id: 'phoenix', name: 'Phoenix Wright', image: 'phoenixwright.png', role: 'lawyer', recurring: true},
    {id: 'young-phoenix', name: 'Young Phoenix', image: 'young-phoenix.png', role: 'defendant', recurring: false, base: 'phoenix'},
    {id: 'edgeworth', name: 'Miles Edgeworth', image: 'edgeworth.png', role: 'lawyer', recurring: true},
    {id: 'young-edgeworth', name: 'Young Edgeworth', image: 'young-edgeworth.png', role: 'lawyer', recurring: false, base: 'edgeworth'},
    {id: 'maya', name: 'Maya Fey', image: 'mayafey.png', role: 'assistant', recurring: true},
    {id: 'redd-white', name: 'Redd White', image: 'reddwhite.png', role: 'witness', recurring: false}
    ...
];

var myPicker = new picker.Picker({
    items: items,
    localStorageKey: 'picker-state',
    defaultSettings: {
        roles: ['lawyer', 'assistant', 'defendant', 'witness', 'other'],
        recurringOnly: true,
        noDuplicates: true
    },
    getFilteredItems: function(settings) {
        var used = {};
        var filteredList = [];
        for (var i = 0; i < items.length; i++) {
            // Skip item if:
            // 1. the character's role is not in the list of roles we've checked, or
            // 2. we checked the "recurring only" box, and the character is not a recurring character.
            if (settings.roles.indexOf(items[i].role) === -1 || settings.recurringOnly && !items[i].recurring) continue;

            // If we want no duplicates, make sure we're not already including a different incarnation of the same character.
            if (settings.noDuplicates) {
                // Skip if we're already including a character with this base.
                if (items[i].base && used[items[i].base]) continue;

                // Record that we're including this character, so we don't introduce duplicates later.
                used[items[i].base || items[i].id] = true;
            }
            filteredList.push(items[i].id);
        }
        return filteredList;
    }
});
```


#### Shortcode Links

The picker also includes built-in support for shareable permalinks to your favorite list that allow anyone to continue a given list from its link. Using this feature is somewhat more involved, but still not too complicated if you use the `picker-shortcodes.html` file as a base over `picker.html`. The steps to get it working (with `picker-shortcodes.html`) are:

1. Give each of your items a unique `shortcode` property. The shortcodes should be short case-sensitive strings of URL-safe characters (sticking with something like letters, numbers, hyphens and underscores, which gives you 64 characters to work with, is a good idea), and they should *all be the same length*. The Favorite Pokémon Picker, with well over a thousand items, gets by on two-character shortcodes - two characters from a 64-character alphabet give you 4096 possibilities - so you probably don't need more than that, but you could also make them slightly longer in order to make them more meaningful.
2. Set the `shortcodeLength` option on the `Picker` to the length of your shortcodes (so, for instance, `2` if you use two-character shortcodes). Optionally, you can also set the `favoritesQueryParam` option to decide the name of the query parameter used for shortcode links (default: `'favs'`).
3. *If you are using custom settings for filtering*, you may want to set the `settingsFromFavorites` option on the `Picker` to a function that takes a list of items and returns any settings that should be set (overriding the default settings) when restoring this set of items. The idea here is that if your default settings filter out some items, then if you visit a shortcode link with some items that would be filtered out by default and choose to continue that list, you probably want to change the settings to permit these items, rather than alter the given list to conform to the settings. For example, supposing you have the settings from the example above, you'd want something like this:

    ```
    var items = [
        {id: 'phoenix', name: 'Phoenix Wright', image: 'phoenixwright.png', role: 'lawyer', recurring: true, shortcode: 'aa'},
        {id: 'edgeworth', name: 'Miles Edgeworth', image: 'edgeworth.png', role: 'lawyer', recurring: true, shortcode: 'ab'},
        {id: 'maya', name: 'Maya Fey', image: 'mayafey.png', role: 'assistant', recurring: true, shortcode: 'ac'},
        {id: 'redd-white', name: 'Redd White', image: 'reddwhite.png', role: 'witness', recurring: false, shortcode: 'ad'}
        ...
    ];

    var myPicker = new picker.Picker({
        items: items,
        localStorageKey: 'picker-state',
        shortcodeLength: 2,
        defaultSettings: {
            roles: ['lawyer', 'assistant', 'defendant', 'witness', 'other'],
            recurringOnly: true
        },
        shouldIncludeItem: function(item, settings) {
            // Include only if:
            // 1. the character's role is in the list of roles we've checked, and
            // 2. we haven't checked the recurring box, *or* the character is recurring.
            return settings.roles.indexOf(item.role) !== -1 && (!settings.recurringOnly || item.recurring);
        },
        settingsFromFavorites: function(favorites) {
            var hasNonRecurring = false;
            for (var i = 0; i < favorites.length; i++) {
                if (!favorites[i].recurring) {
                    hasNonRecurring = true;
                }
            }
            return {
                recurringOnly: !hasNonRecurring
            };
        }
    });
    ```

And that's about it - `picker-shortcodes.html` will handle the rest. Once implemented, there will be an automatically updating link below the current favorite list, and visiting that link will pop up a modal window showing the shared list as well as buttons allowing the user to continue from that list or dismiss it.

**Keep in mind that due to the permanent nature of links, shortcodes should never be changed.** Once you make a public version of your picker where certain items have certain shortcodes, please ensure that in the future you only *add* shortcodes, or perhaps remove some if truly appropriate. Altering the picker so that a shortcode that used to refer to one item now refers to another will **change what existing links point to**! If your picker will foreseeably gain more items in the future (for example, the Favorite Pokémon Picker is updated whenever new Pokémon are added in new installments of the video game series), ensure the way you assign shortcodes is future-proof and won't require you to devise a different scheme later.


## Advanced Customization

If you're comfortable writing significant JavaScript, there are a myriad more ways to customize and work with the picker.

The picker's code is split into two [UMD](https://github.com/umdjs/umd) modules, `picker.js` (which contains all the actual logic and functionality of the picker) and `picker-ui.js` (a flexible jQuery-based user interface for the picker). This split means you can easily do away with `picker-ui.js` altogether and build your own UI with no jQuery dependency. For example, if you'd like to make a [React](https://reactjs.org) interface for the picker, you can do that easily:

```
var picker = require('./picker.js');

class PickerContainer extends React.Component {
    state = {
        picker: new picker.Picker({
            ...
        })
    }

    render() {
        var self = this;
        var picker = this.state.picker;
        var wrapAction = func => function () {
            func.apply(picker, arguments);
            self.setState({ picker: picker });
        };
        return (
            <Picker
                evaluating={picker.getEvaluating()}
                favorites={picker.getFavorites()}
                settings={picker.getSettings()}
                onPick={wrapAction(picker.pick)}
                onPass={wrapAction(picker.pass)}
                onUndo={wrapAction(picker.undo)}
                onRedo={wrapAction(picker.redo)}
                onChangeSettings={wrapAction(picker.setSettings)}
            />
        );
    }
}
```

Then all you have to do is build a `Picker` component rendering your desired UI based on the props passed to it by the `PickerContainer`. (Note that you will have to, for example, keep track of the currently selected items separately.) Refer to the API reference below for additional properties and methods you can pass down to your Picker component.


## API reference


### `picker.PickerState` (module `picker.js`)

The PickerState object represents the state of the picker's data, handles the algorithmic logic behind how it changes on user actions, and ensures the validity of the state. While for standard pickers meant to be used in-browser you should use the Picker object, which adds a lot of features around the basic state logic, you can use the PickerState object directly if you plan to do something more customized - if you're running a picker on the server side, say - and only want the picker logic itself.


#### `picker.PickerState(options)`

Constructs and returns a PickerState. The options object specifies options for the PickerState:

- `items` (array): **Required**. A plain list of all valid item identifiers (IDs).
- `getBatchSize` (`function(currentSize, settings)`): A function that takes the current number of items in this round and the current settings and returns the number of items that should be displayed in each batch of items. By default this is the `currentSize` divided by five, clamped by the `minBatchSize` and `maxBatchSize` settings.
- `shouldIncludeItem` (`function(identifier, settings)`): A function that takes an item identifier and the current settings and returns a truthy value if that item should be included given these settings. By default, all items are always included.
- `getFilteredItems` (`function(settings)`): A function that takes the current settings and returns an array of item identifiers that should be included given these settings. By default, all items are included. (Specifying this will override any `shouldIncludeItem` function.)
- `defaultSettings` (object): An object of default settings (see the settings part of the documentation).


#### PickerState public methods

- `getState()`: Returns a *dehydrated state* representing this PickerState. This is a plain JavaScript object with the following properties:
  - `eliminated` (array): A list of objects of the form `{id: 'identifier', eliminatedBy: ['a', 'b']}`, where the `id` is an item identifier and `eliminatedBy` is an array of item identifiers that were picked over this item.
  - `survived` (array): A list of item identifiers that have survived this round so far (i.e. were picked from their respective batches).
  - `current` (array): A list of item identifiers that have yet to be presented to the user this round.
  - `evaluating` (array): A list of item identifiers in the batch of items currently being evaluated.
  - `favorites` (array): A list of item identifiers that have been discovered as favorites, in order.
  - `settings` (object): The current picker settings.
- `initialize(settings)`: Initializes the PickerState with the given settings.
- `restoreState(state)`: Takes a *dehydrated state* and restores it into the current state.
- `reset()`: Resets the PickerState, re-initializing it with the current settings.
- `setSettings(settings)`: Sets new settings for the state, while ensuring the continued integrity of the state.
- `setFavorites(favorites)`: Takes a list of item identifiers and overwrites the `favorites` list with them, while ensuring the continued integrity of the state.
- `getBatchSize(currentSize)`: Takes the size of a round and returns the appropriate batch size.
- `resetBatchSize()`: Sets the current batch size to whatever the current batch size should be given the size of the current round and adjusts the evaluating batch accordingly.
- `pick(picked)`: Picks a list of items. `picked` should be an array of item identifiers from the current evaluating batch, which the user has picked over the rest of the items in the batch. There is technically nothing ensuring these items are actually from the evaluating batch, so you could do some funky things with it, but this is highly disrecommended unless you really know what you're doing algorithmically.
- `pass()`: Passes on the current evaluating batch. This is equivalent to picking every item in the batch.


### `picker.Picker` (module `picker.js`)

The Picker object wraps the PickerState and provides various additional functionality to enhance it: an undo history, automatically saving the current state after user actions and loading the current state on startup, and the shortcode link functionality described above.


#### `picker.Picker(options)`

Constructs and returns a Picker. The options object specifies options for the Picker:

- `items` (array): **Required**. An array of item objects with at least an `id` property, and possibly `name` and `image` properties and any custom properties required for this particular Picker.
- `localStorageKey`: A string specifying the key used for the favorite state in the browser's localStorage. Set this to a string such as 'picker-state' in order to automatically store the picker state in the user's browser under this key. (The default is to not store it, since otherwise the picker might inadvertently clobber a localStorage key already in use on the host website.)
- `defaultSettings` (object): An object of default settings for the Picker. May include a `minBatchSize` and/or `maxBatchSize`, as well as any custom settings used for this Picker.
- `historyLength` (integer): An integer specifying how many history entries should be maintained for undo/redo purposes. (Default: `3`)
- `getBatchSize` (`function(currentSize, settings)`): A function that takes the current number of items in this round and the current settings and returns the number of items that should be displayed in each batch of items. By default this is the `currentSize` divided by five, clamped by the `minBatchSize` and `maxBatchSize` settings.
- `shouldIncludeItem` (`function(item, settings)`): A function that takes an item object and the current settings and returns a truthy value if that item should be included given these settings. By default, all items are included.
- `getFilteredItems` (`function(settings)`): A function that takes the current settings and returns an array of item identifiers that should be included given these settings. By default, all items are included. (Specifying this will override any `shouldIncludeItem` function.)
- `modifyState` (`function(state)`): A function that takes a dehydrated state that we want to restore and returns a transformed version of the state. Useful for backwards-compatibility purposes - if the user has an outdated state saved in their browser, this hook allows you to alter the state or even replace it entirely before it's restored. By default, the unmodified state is restored, provided it's a valid state.
- `onLoadState` (`function(missingItems, extraItems)`): A function called when a saved state has just been loaded, with the `Picker` object as `this`. This hook allows you to do any work that needs to be done after a state is restored. For example, the Favorite Pokémon Picker uses this hook to display a "Welcome back" message on the page explaining that the previous state has been loaded. `missingItems` and `extraItems` are arrays containing any item identifiers not present in the loaded state even though they ought to be there according to the options passed to the picker, and any item identifiers present in the loaded state even though they shouldn't be there, respectively.

  In practice, unless your users are manually messing with their state, these parameters will come into play only if you've edited the picker since the user last used it, such as by adding more items or removing some items. The picker will figure it out and silently edit the state to match with the current version of the picker, but these parameters allow you to do something with this information, such as informing the user that items have been added/removed.
- `saveState` (`function(state)`): A function that takes a dehydrated state and saves it, used to override the default localStorage save feature.
- `loadState` (`function()`): A function that retrieves and returns a dehydrated state that should be loaded for the user, used to override the default localStorage save feature. This is only run once, when first initializing the picker; if you need to do something like fetching saved state data asynchronously, then construct the Picker object *after* you've fetched the data:
  ```
  fetchMySavedState(function (err, loadedState) {
      var myPicker = picker.Picker({
          items: ...,
          saveState: function (state) {
              saveMyState(state);
          },
          loadState: function() {
              return loadedState;
          }
      });
  });
  ```
- `favoritesQueryParam`: A string specifying a query parameter used to specify a shortcode string for a favorite list (see the shortcode section of the documentation). (Default: `'favs'`)
- `shortcodeLength`: The number of characters used for each item's shortcode (see the shortcode section of the documentation).
- `settingsFromFavorites` (`function(favorites)`): A function that takes a list of items and returns a settings object representing those settings that should be assumed when this favorite list is restored through a shortcode string (see the shortcode section of the documentation).


#### Picker public methods

- `getFavorites()`: Gets the list of items on the current found favorites list.
- `getEvaluating()`: Gets the list of items currently being evaluated.
- `getSettings()`: Gets the current settings of the picker.
- `getSharedFavorites()`: Gets the favorite list shared by this shortcode link (see the shortcode section of the documentation).
- `getShortcodeString()`: Gets the shortcode string for the current favorite list (see the shortcode section of the documentation).
- `getShortcodeLink()`: Gets the full relative shortcode link, based on the `favoritesQueryParam` option and the `getShortcodeString()` method (see the shortcode section of the documentation).
- `parseShortcodeString(shortcodeString)`: Takes a shortcode string and parses it into a list of item identifiers (see the shortcode section of the documentation).
- `pushHistory()`: Pushes the current state into the undo history and saves it.
- `canUndo()`: Returns `true` if you can undo (i.e. if there is a previous history entry), `false` otherwise.
- `canRedo()`: Returns `true` if you can redo (i.e. if there is a next history entry), `false` otherwise.
- `undo()`: Undoes the previous action, returning to the previous state.
- `redo()`: Redoes a previously undone action, returning to that state.
- `resetToFavorites(favorites, useSettings)`: Sets the state to a clean slate with just the given favorites listed in Found Favorites. `useSettings` can be a settings object, in which case that settings object is set and used to determine if a favorite given by the list should be kept or discarded, according to the `shouldIncludeItem` function. Otherwise, however, the settings will be set according to the `settingsFromFavorites` function.
- `saveState()`: Saves the current state. By default, this is a noop unless the `localStorageKey` option is specified, in which case the state will be saved to localStorage under that key; however, if a `saveState` function is provided in the options, that will be run instead.
- `loadState()`: Loads and returns a dehydrated state. By default, if the `localStorageKey` option is specified, it'll retrieve the state from there; otherwise, it returns `undefined`. However, if a `loadState` function is provided in the options, it will return the result of that instead.
- `isUntouched()`: Returns `true` if the picker state is untouched - that is, if the user has not made any picks or passes.
- `hasItems()`: Returns `true` if the *filtered* list of items - that is, the list of items for which the provided `shouldIncludeItem` function returns a truthy value - has at least one item in it.
- `pick(picked)`: Takes a list of item identifiers and picks them from the current evaluating batch. (See the `pick` method of `PickerState`.)
- `pass()`: Passes on this batch of items. This is equivalent to picking every item in the batch.
- `reset()`: Resets the picker.
- `setSettings(settings)`: Sets the picker settings and re-filters items accordingly.
- `setFavorites(favorites)`: Takes a list of item identifiers and overwrites the `favorites` list with them, while ensuring the continued integrity of the state.


### `PickerUI` (module `picker-ui.js`)

This separate module contains a full-featured UI for the picker, which depends on [jQuery](https://jquery.com). If you'd like to build your own UI using a different UI library or vanilla JavaScript, you can skip `picker-ui.js` altogether and just call methods on your `Picker` objects yourself from your own code.

Otherwise, the provided PickerUI should be pretty easy to use, particularly with the default `picker.html` file (or `picker-shortcodes.html`) but also if you build your own custom layout.


#### `PickerUI(picker, options)`

Constructs and returns a PickerUI. The `picker` parameter should be a `Picker` object. The `options` parameter is an object specifying additional options for the picker UI:

- `elements` (object): **Required.** An object mapping predefined element names to jQuery selectors for those elements. You don't need to touch this if you're using the provided `picker.html` or `picker-shortcodes.html`. The elements are:
  - `pick`: The "Pick" button.
  - `pass`: The "Pass" button.
  - `undo`: The "Undo" button.
  - `redo`: The "Redo" button.
  - `evaluating`: The container for the current evaluating batch (should be a `ul` element, unless you set the `wrapItem` function).
  - `favorites`: The container for the found favorites (should be a `ul` or `ol` element, unless you set the `wrapItem` function).
  - `reset`: **Optional**. Any extra static reset button(s) you may have placed in your HTML.
  - `shortcodeLink`: **Optional**. The shortcode link (only used if the `Picker` object has the `favoritesQueryParam` and `shortcodeLength` options set).
  - `sharedList`: **Optional**. The container for the shared list for this shortcode link (should be a `ul` or `ol` element, unless you set the `wrapItem` function) (see the shortcode part of the documentation).
  - `sharedListContainer`: **Optional**. The outer container (e.g. a modal window) for a shared list displayed when visiting a shortcode link (see the shortcode section of the documentation).
  - `sharedListContinue`: **Optional**. A button that continues from the shared list given by this shortcode link (see the shortcode section of the documentation).
  - `sharedListSkip`: **Optional**. Any buttons to dismiss a shared list given by a shortcode link (see the shortcode section of the documentation).
  - `settings`: A sub-object mapping setting names to jQuery selectors for setting elements (see the settings part of the documentation).
- `messages` (object): Optionally, if you would like to override some or all of the messages displayed in the picker - such as to display them in a different language - you can specify them here. The messages are:
  - `reset`: The default text of generated reset buttons. By default, this is "Reset".
  - `mustSelect`: The alert shown to the user if they try to pick without having selected anything. By default, this is "You must select something first! If you're indifferent, press Pass."
  - `orderedAll`: The message displayed when the user has ordered every included item. By default, this is "You have ordered every available item!"
  - `noItems`: The message displayed if the user has selected settings such that every item is filtered out. By default, this is "There are no items that fit your criteria! Set some different options and try again."
  - `resetWarning`: This is a general message prompting the user for whether they would like to reset their state. By default, this is "Are you sure you wish to reset your state? All your found favorites and current progress will be lost."
- `onUpdate` (`function()`): A function to be called whenever the UI has updated. `this` will be set to the `PickerUI` object, from which you can also access `this.picker` and `this.picker.state`. The Favorite Pokémon Picker uses this function to display the number of Pokémon remaining in this round, a dump of the current dehydrated state for debugging purposes, and more.
- `getItemImageUrl` (`function(item, settings)`): By default, the picker will use the `image` property of the item as the image source if it's present. However, if you'd rather derive an image URL from other properties of the item and/or the current settings, you can specify this function.
- `wrapItem` (`function(itemContent)`): A function that takes the contents representing a given item and wraps them in a container element, returning the container (as an element or jQuery object). By default, it wraps the item in a `li` element; if you need it to be something else, like a `div`, or want to do extra wrapping for styling purposes, overriding this is a lot simpler than overriding `getItemElem` altogether.
- `getItemElem` (`function(item, settings)`): If `wrapItem` and `getItemImageUrl` aren't enough, you can write your own function taking an item object and the current settings and returning an element or jQuery object representing this item.


#### PickerUI public methods

- `initialize()`: Initializes the picker UI, setting any setting elements and performing the first UI update.
- `getSetting(setting)`: Gets the current value of a setting element (see the settings part of the documentation).
- `setSetting(setting, value)`: Sets the value of a setting element (see the settings part of the documentation).
- `getSettings()`: Gets a settings object from all defined setting elements (see the settings part of the documentation).
- `setSettings(settings)`: Takes a setting object and sets the specified setting elements accordingly.
- `getSelected()`: Gets the list of items currently selected.
- `update()`: Updates the UI according to the current picker state.
- `pick(items)`: Picks the given item identifiers and updates the UI accordingly.
- `pass()`: Passes on this batch and updates the UI accordingly.
- `undo()`: Undoes the last action and updates the UI accordingly.
- `redo()`: Redoes the previously undone action and updates the UI accordingly.
- `reset()`: Prompts the user for a reset and, if they confirm, resets the picker and updates the UI accordingly.
- `displaySharedList()`: Shows the shared list container.
- `dismissSharedList()`: Hides the shared list container.
- `getItemElem(item, settings)`: Returns a jQuery object for an element representing the given item with these settings.
- `makeResetButton(text)`: Creates and returns a jQuery object for a reset button with the given text, which will prompt the user for a reset when pressed. Use if you'd like to conditionally add a reset button to the UI somewhere.


The source code of `picker.js` and `picker-ui.js` contains various other methods less intended for public use - feel free to have a scroll through if the functionality you want isn't documented here.
