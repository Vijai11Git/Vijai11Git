import React, { useState, useEffect } from "react";

const Section = ({ section, onEdit, onDelete, onClone, onToggle }) => (
  <div>
    <div
      style={{
        cursor: "pointer",
        fontWeight: "bold",
        marginBottom: "0.5rem"
      }}
      onClick={() => onToggle(section.id)}
    >
      {section.expanded ? "-" : "+"} {section.accordionTitle}
    </div>
    {section.expanded && (
      <div>
        <div>{section.accordionDescription}</div>
        <button onClick={() => onEdit(section.id)}>Edit</button>
        <button onClick={() => onDelete(section.id)}>Delete</button>
        <button onClick={() => onClone(section.id)}>Clone</button>
      </div>
    )}
  </div>
);

const GroupList = ({
  groups,
  onGroupEdit,
  onGroupDelete,
  onGroupPreview,
  onGroupClone
}) => {
  return (
    <div>
      <h3>Group List</h3>
      <ul>
        {Object.keys(groups).map((groupName) => (
          <li key={groupName}>
            {groupName}{" "}
            <button onClick={() => onGroupEdit(groupName)}>Edit</button>
            <button onClick={() => onGroupDelete(groupName)}>Delete</button>
            <button onClick={() => onGroupPreview(groupName)}>Preview</button>
            <button onClick={() => onGroupClone(groupName)}>Clone</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

const ExpandCollapse = () => {
  const [groupTitle, setGroupTitle] = useState("");
  const [groupDescription, setGroupDescription] = useState("");
  const [accordionTitle, setAccordionTitle] = useState("");
  const [accordionDescription, setAccordionDescription] = useState("");
  const [sections, setSections] = useState([]);
  const [allGroups, setAllGroups] = useState({});
  const [editingSectionId, setEditingSectionId] = useState(null);
  const [editingGroupId, setEditingGroupId] = useState(null);
  const [editedGroupTitle, setEditedGroupTitle] = useState(null);
  const [selectedGroup, setSelectedGroup] = useState(null);

  useEffect(() => {
    // Load data from local storage if available
    const storedGroups = localStorage.getItem("groups");
    if (storedGroups) {
      setAllGroups(JSON.parse(storedGroups));
    }
  }, []);

  useEffect(() => {
    // Save data to local storage when allGroups is updated
    localStorage.setItem("groups", JSON.stringify(allGroups));
  }, [allGroups]);

  const handleAddSection = () => {
    if (accordionTitle.trim() === "" || accordionDescription.trim() === "") {
      alert("Please enter accordion title and description.");
      return;
    }

    if (editingSectionId !== null) {
      // Editing an existing section
      const updatedSections = sections.map((section) =>
        section.id === editingSectionId
          ? { ...section, accordionTitle, accordionDescription }
          : section
      );
      setSections(updatedSections);
      setEditingSectionId(null); // Reset the editing section ID
    } else {
      // Adding a new section
      const newSection = {
        id: Date.now(), // Generate a unique ID for each new section
        accordionTitle,
        accordionDescription,
        expanded: false
      };

      setSections((prevSections) => [...prevSections, newSection]);
    }

    setAccordionTitle("");
    setAccordionDescription("");
  };

  const handleToggleSection = (sectionId) => {
    setSections((prevSections) =>
      prevSections.map((section) =>
        section.id === sectionId
          ? { ...section, expanded: !section.expanded }
          : section
      )
    );
  };

  const handleAddGroup = () => {
    if (
      groupTitle.trim() === "" ||
      groupDescription.trim() === "" ||
      sections.length === 0
    ) {
      alert(
        "Please enter group title, description, and add at least one section."
      );
      return;
    }

    const newGroup = {
      id: Date.now(),
      groupTitle,
      groupDescription,
      sections
    };

    setAllGroups((prevAllGroups) => ({
      ...prevAllGroups,
      [groupTitle]: [...(prevAllGroups[groupTitle] || []), newGroup]
    }));

    setGroupTitle("");
    setGroupDescription("");
    setSections([]);
  };

  const handleEditSection = (sectionId) => {
    setEditingSectionId(sectionId);
    const sectionToEdit = sections.find((section) => section.id === sectionId);
    if (!sectionToEdit) return;

    setAccordionTitle(sectionToEdit.accordionTitle);
    setAccordionDescription(sectionToEdit.accordionDescription);
  };

  const handleDeleteSection = (sectionId) => {
    setSections((prevSections) =>
      prevSections.filter((section) => section.id !== sectionId)
    );
  };

  const handleCloneSection = (sectionId) => {
    const sectionToClone = sections.find((section) => section.id === sectionId);
    if (!sectionToClone) return;

    const newSection = {
      ...sectionToClone,
      id: Date.now()
    };

    setSections((prevSections) => [...prevSections, newSection]);
  };

  const handleEditGroup = (groupName) => {
    setGroupTitle(groupName);
    setGroupDescription(allGroups[groupName][0].groupDescription);
    setSections(allGroups[groupName][0].sections);
    setEditingGroupId(groupName);
    setEditedGroupTitle(groupName);
  };

  const handleGroupDelete = (groupName) => {
    setAllGroups((prevAllGroups) => {
      const updatedGroups = { ...prevAllGroups };
      delete updatedGroups[groupName];
      return updatedGroups;
    });
  };

  const handleGroupPreview = (groupName) => {
    setSelectedGroup(allGroups[groupName][0]);
  };

  const handleGroupClone = (groupName) => {
    const groupToClone = allGroups[groupName];
    if (!groupToClone) return;

    const clonedGroup = groupToClone.map((group) => {
      const newSections = group.sections.map((section) => ({
        ...section,
        id: Date.now() // Assign a new unique ID to each cloned section
      }));

      return {
        ...group,
        id: Date.now(), // Assign a new unique ID to the cloned group
        sections: newSections
      };
    });

    // Generate a new unique key for the cloned group
    const newGroupName = groupName + "_clone_" + Date.now();

    setAllGroups((prevAllGroups) => ({
      ...prevAllGroups,
      [newGroupName]: clonedGroup // Store the cloned group using the new unique key
    }));
  };

  const updateGroupName = (oldGroupName, newGroupName) => {
    setAllGroups((prevAllGroups) => {
      const updatedGroups = { ...prevAllGroups };
      updatedGroups[newGroupName] = updatedGroups[oldGroupName];
      delete updatedGroups[oldGroupName];
      return updatedGroups;
    });
  };

  const handleSaveAll = () => {
    if (
      groupTitle.trim() === "" ||
      groupDescription.trim() === "" ||
      sections.length === 0
    ) {
      alert(
        "Please enter group title, description, and add at least one section."
      );
      return;
    }

    if (editingGroupId !== null) {
      // Updating an existing group
      updateGroupName(editingGroupId, groupTitle);
      setEditedGroupTitle(groupTitle);

      if (selectedGroup && selectedGroup.groupTitle === editingGroupId) {
        setSelectedGroup({
          ...selectedGroup,
          groupTitle,
          groupDescription,
          sections
        });
      } else {
        // Adding a new group
        const newGroup = {
          id: Date.now(),
          groupTitle,
          groupDescription,
          sections
        };

        setAllGroups((prevAllGroups) => ({
          ...prevAllGroups,
          [groupTitle]: [...(prevAllGroups[groupTitle] || []), newGroup]
        }));
      }

      setGroupTitle("");
      setEditedGroupTitle(""); // Reset the editedGroupTitle
      setGroupDescription("");
      setSections([]);
      setEditingGroupId(null); // Reset the editing group ID after saving
    }
  };

  return (
    <div>
      <div>
        <input
          type="text"
          value={groupTitle}
          onChange={(e) => setGroupTitle(e.target.value)}
          placeholder="Group Title"
        />
        <textarea
          value={groupDescription}
          onChange={(e) => setGroupDescription(e.target.value)}
          placeholder="Group Description"
        />
      </div>

      <div>
        <input
          type="text"
          value={accordionTitle}
          onChange={(e) => setAccordionTitle(e.target.value)}
          placeholder="Accordion Title"
        />
        <textarea
          value={accordionDescription}
          onChange={(e) => setAccordionDescription(e.target.value)}
          placeholder="Accordion Description"
        />
        <button onClick={handleAddSection}>Add Section</button>
      </div>

      {sections.length > 0 && (
        <div>
          {sections.map((section) => (
            <Section
              key={section.id}
              section={section}
              onEdit={handleEditSection}
              onDelete={handleDeleteSection}
              onClone={handleCloneSection}
              onToggle={handleToggleSection}
            />
          ))}
        </div>
      )}

      <button onClick={handleAddGroup}>Add Group</button>

      <GroupList
        groups={allGroups}
        onGroupEdit={handleEditGroup}
        onGroupDelete={handleGroupDelete}
        onGroupPreview={handleGroupPreview}
        onGroupClone={handleGroupClone}
      />
      <button onClick={handleSaveAll}>Save All</button>

      {/* Render the group preview */}
      {selectedGroup && (
        <div>
          <h3>Group Preview</h3>
          <div>
            <h4>Group Title: {editedGroupTitle}</h4>
            <p>Group Description: {selectedGroup.groupDescription}</p>
            {selectedGroup.sections.map((section) => (
              <div key={section.id}>
                <h5>Accordion Title: {section.accordionTitle}</h5>
                <p>Accordion Description: {section.accordionDescription}</p>
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
};

export default ExpandCollapse;
