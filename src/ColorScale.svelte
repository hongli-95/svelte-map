<script>
    //WIP
    import { scaleLinear } from "d3-scale";
    import { extent } from "d3-array";
    import { json } from "d3";

    let dataset = [];
    export let idList = [];
    let colorScale = () => {};

    json(
		'https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.json'
	).then((dataDump) => {
		dataset = dataDump;
        idList = dataDump.iso_code;

        let popExtent = extent()
    })

    

</script>