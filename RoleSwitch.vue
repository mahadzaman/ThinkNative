<template>
    <div id = "cpanel-switch" class = "control-panel">
        <div class = "backdrop"></div>
        <div class = "control-panel-wrapper">
            <div class = "select-company-wrapper mar-b-10">
                <p class = "mar-b-5">Select Company</p>
                <div class = "custom-select">
                    <span class = "custom-selected">Add Note</span>
                    <ul class = "custom-options">
                        <li class = "c-option c-selected">
                            <a href = "#">Company A</a>
                        </li>
                        <li class = "c-option">
                            <a href = "#">Company B</a>
                        </li>
                        <li class = "c-option">
                            <a href = "#">Company C</a>
                        </li>
                        <li class = "c-option">
                            <a href = "#">Company D</a>
                        </li>
                        <li class = "c-option">
                            <a href = "#">Company E</a>
                        </li>
                        <li class = "c-option">
                            <a href = "#">Company F</a>
                        </li>
                        <li class = "c-option">
                            <a href = "#">Company G</a>
                        </li>
                    </ul>
                </div>
            </div>
            <div class = "role-toggle-wrapper">
                <p class = "mar-b-5">Choose active Role</p>
                <div class = "role-toggle-container">
                    <input name = "role" type = "radio" checked href = "#" class = "member" value = "member" />
                    <input name = "role" type = "radio" href = "#" class = "champion" value = "champion" />
                </div>
            </div>
            <a href = "#" class = "toggle-panel">
                <i class = "icon-close"></i>
                <i class = "icon-gear"></i>
            </a>
        </div>
    </div>
</template>


<script>

    export default {
        name : "role-switch",
    };
</script>

