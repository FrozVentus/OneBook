# frozventus-reused
###### /java/seedu/address/ui/PersonDisplayCard.java
``` java
    private void loadPersonDetails(ReadOnlyPerson person) {
        this.person = person;
        initTags(person);
        bindListeners(person);
    }

```
###### /java/seedu/address/ui/PersonDisplayCard.java
``` java
    /**
     * Binds the individual UI elements to observe their respective {@code Person} properties
     * so that they will be notified of any changes.
     */
    private void bindListeners(ReadOnlyPerson person) {
        nameLarge.textProperty().bind(Bindings.convert(person.nameProperty()));
        phoneLarge.textProperty().bind(Bindings.convert(person.phoneProperty()));
        birthdayLarge.textProperty().bind(Bindings.convert(person.birthdayProperty()));
        emailLarge.textProperty().bind(Bindings.convert(person.emailProperty()));
        organisationLarge.textProperty().bind(Bindings.convert(person.organisationProperty()));
        remarkLarge.textProperty().bind(Bindings.convert(person.remarkProperty()));
        person.tagProperty().addListener((observable, oldValue, newValue) -> {
            tagsLarge.getChildren().clear();
            person.getTags().forEach(tag -> tagsLarge.getChildren().add(new Label(tag.tagName)));
        });
    }

```
###### /java/seedu/address/ui/PersonDisplayCard.java
``` java
    private void initTags(ReadOnlyPerson person) {
        tagsLarge.getChildren().clear();
        person.getTags().forEach(tag -> tagsLarge.getChildren().add(new Label(tag.tagName)));
    }

```
###### /java/seedu/address/ui/PersonCard.java
``` java
    /**
     * Binds the individual UI elements to observe their respective {@code Person} properties
     * so that they will be notified of any changes.
     */
    private void bindListeners(ReadOnlyPerson person) {
        name.textProperty().bind(Bindings.convert(person.nameProperty()));
        phone.textProperty().bind(Bindings.convert(person.phoneProperty()));
        person.tagProperty().addListener((observable, oldValue, newValue) -> {
            tags.getChildren().clear();
            person.getTags().forEach(tag -> tags.getChildren().add(new Label(tag.tagName)));
        });
    }

    private void initTags(ReadOnlyPerson person) {
        person.getTags().forEach(tag -> tags.getChildren().add(new Label(tag.tagName)));
    }

    @Override
    public boolean equals(Object other) {
        // short circuit if same object
        if (other == this) {
            return true;
        }

        // instanceof handles nulls
        if (!(other instanceof PersonCard)) {
            return false;
        }

        // state check
        PersonCard card = (PersonCard) other;
        return id.getText().equals(card.id.getText())
                && person.equals(card.person);
    }
}
```
###### /java/seedu/address/logic/parser/BinDeleteCommandParser.java
``` java
    /**
     * Parses the given {@code String} of arguments in the context of the DeleteCommand
     * and returns an DeleteCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public BinDeleteCommand parse(String args) throws ParseException {
        try {
            Index[] index = ParserUtil.parseDeleteIndex(args);
            return new BinDeleteCommand(index);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, BinDeleteCommand.MESSAGE_USAGE));
        }
    }

}
```
###### /java/seedu/address/logic/parser/RestoreCommandParser.java
``` java
    /**
     * Parses the given {@code String} of arguments in the context of the DeleteCommand
     * and returns an DeleteCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public RestoreCommand parse(String args) throws ParseException {
        try {
            Index[] index = ParserUtil.parseDeleteIndex(args);
            return new RestoreCommand(index);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, RestoreCommand.MESSAGE_USAGE));
        }
    }

}
```
###### /java/seedu/address/logic/commands/RestoreCommand.java
``` java
    @Override
    public CommandResult executeUndoableCommand() throws CommandException {

        List<ReadOnlyPerson> lastShownList = model.getRecycleBinList();
        for (Index targetIndex : targetIndex) {
            if (targetIndex.getZeroBased() >= lastShownList.size()) {
                throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
            }
        }

        if (targetIndex.length > 1) {
            for (int i = 1; i < targetIndex.length; i++) {
                if (targetIndex[i].getZeroBased() < targetIndex[i - 1].getZeroBased()) {
                    throw new CommandException(Messages.MESSAGE_INVALID_ORDER_PERSONS_INDEX);
                } else if (targetIndex[i].getZeroBased() == targetIndex[i - 1].getZeroBased()) {
                    throw new CommandException(Messages.MESSAGE_REPEATED_INDEXES);
                }
            }
        }

        ReadOnlyPerson personToRestore;
        String[] personRestoredMessage = new String[targetIndex.length];
        StringBuilder restoreMessage = new StringBuilder();

        for (int i = (targetIndex.length - 1); i >= 0; i--) {
            int target = targetIndex[i].getZeroBased();
            personToRestore = lastShownList.get(target);
            try {
                model.restorePerson(personToRestore);
            } catch (PersonNotFoundException pnfe) {
                assert false : "The target person cannot be missing";
            } catch (DuplicatePersonException dpe) {
                throw new CommandException(AddCommand.MESSAGE_DUPLICATE_PERSON);
            }
            personRestoredMessage[i] = MESSAGE_RESTORE_PERSON_SUCCESS + personToRestore;
        }

        for (String message : personRestoredMessage) {
            restoreMessage.append(message);
            restoreMessage.append("\n");
        }

        return new CommandResult(restoreMessage.toString().trim());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof RestoreCommand // instanceof handles nulls
                && Arrays.equals(this.targetIndex, ((RestoreCommand) other).targetIndex)); // state check
    }
}
```
###### /java/seedu/address/logic/commands/Command.java
``` java
    /**
     * Provides any needed dependencies to the command.
     * Commands making use of any of these should override this method to gain
     * access to the dependencies.
     */
    public void setData(Model model, CommandHistory history, UndoRedoStack undoRedoStack, boolean binMode) {
        this.model = model;
        this.binMode = binMode;
    }
}
```
###### /java/seedu/address/logic/commands/BinDeleteCommand.java
``` java
    @Override
    public CommandResult executeUndoableCommand() throws CommandException {

        List<ReadOnlyPerson> lastShownList = model.getRecycleBinList();
        for (Index targetIndex : targetIndex) {
            if (targetIndex.getZeroBased() >= lastShownList.size()) {
                throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
            }
        }

        if (targetIndex.length > 1) {
            for (int i = 1; i < targetIndex.length; i++) {
                if (targetIndex[i].getZeroBased() < targetIndex[i - 1].getZeroBased()) {
                    throw new CommandException(Messages.MESSAGE_INVALID_ORDER_PERSONS_INDEX);
                } else if (targetIndex[i].getZeroBased() == targetIndex[i - 1].getZeroBased()) {
                    throw new CommandException(Messages.MESSAGE_REPEATED_INDEXES);
                }
            }
        }

        ReadOnlyPerson personToDelete;
        String[] personDeletedMessage = new String[targetIndex.length];
        StringBuilder deleteMessage = new StringBuilder();

        for (int i = (targetIndex.length - 1); i >= 0; i--) {
            int target = targetIndex[i].getZeroBased();
            personToDelete = lastShownList.get(target);
            try {
                model.deleteFromBin(personToDelete);
            } catch (PersonNotFoundException pnfe) {
                assert false : "The target person cannot be missing";
            }
            personDeletedMessage[i] = MESSAGE_DELETE_PERSON_SUCCESS + personToDelete;
        }

        for (String message : personDeletedMessage) {
            deleteMessage.append(message);
            deleteMessage.append("\n");
        }

        return new CommandResult(deleteMessage.toString().trim());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof BinDeleteCommand // instanceof handles nulls
                && Arrays.equals(this.targetIndex, ((BinDeleteCommand) other).targetIndex)); // state check
    }
}
```
###### /java/seedu/address/logic/commands/BinListCommand.java
``` java
    @Override
    public CommandResult execute() {
        EventsCenter.getInstance().post(new DisplayBinEvent());
        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(MESSAGE_SUCCESS);
    }
}
```
###### /java/seedu/address/model/ModelManager.java
``` java
    @Override
    public void executeBinSort(String sortType) throws EmptyAddressBookException {
        recycleBin.executeSort(sortType);
        indicateAddressBookChanged();
    }

```
###### /java/seedu/address/model/Model.java
``` java
    void executeBinSort(String sortType) throws EmptyAddressBookException;

```
