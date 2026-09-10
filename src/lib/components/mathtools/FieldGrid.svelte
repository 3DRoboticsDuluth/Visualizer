<script lang="ts">
  import { coordinateSettings } from "../../../stores";
  import {
    toDisplay,
    gridOrigin,
    gridSpacing,
    unitLabel,
  } from "../../../utils/coordinates";
  import { gridSize } from "../../../stores";
  import { FIELD_SIZE } from "../../../config";
  import type * as d3 from "d3";

  interface Props {
    x: d3.ScaleLinear<number, number>;
    y: d3.ScaleLinear<number, number>;
  }

  let { x, y }: Props = $props();

  let spacing = $derived(
    Math.max(0.5, gridSpacing($gridSize || 12, $coordinateSettings)),
  );

  let gridPositions = $derived(
    (() => {
      const positions: number[] = [0];
      const origin = gridOrigin($coordinateSettings);
      const first = origin - Math.floor(origin / spacing) * spacing;
      for (
        let pos = first > 0.000001 ? first : spacing;
        pos < FIELD_SIZE;
        pos += spacing
      ) {
        positions.push(Number(pos.toFixed(6)));
      }
      if (positions[positions.length - 1] !== FIELD_SIZE) {
        positions.push(FIELD_SIZE);
      }
      return positions;
    })(),
  );

  // Adjust label frequency and size based on grid density
  let labelInterval = $derived(
    spacing <= 1 ? 12 : spacing <= 3 ? 4 : spacing <= 6 ? 2 : 1,
  );
  let labelFontSize = $derived(
    spacing <= 1
      ? "text-[8px]"
      : spacing <= 3
        ? "text-[9px]"
        : spacing <= 6
          ? "text-[10px]"
          : "text-xs",
  );

  function showsLabel(position: number): boolean {
    return (
      Math.abs(
        (position - gridOrigin($coordinateSettings)) / spacing / labelInterval -
          Math.round(
            (position - gridOrigin($coordinateSettings)) /
              spacing /
              labelInterval,
          ),
      ) < 0.00001 ||
      position === 0 ||
      position === FIELD_SIZE
    );
  }
</script>

<svg class="absolute top-0 left-0 w-full h-full pointer-events-none z-20">
  <!-- Vertical grid lines -->
  {#each gridPositions as position, i (position)}
    <line
      x1={x(position)}
      y1={y(0)}
      x2={x(position)}
      y2={y(FIELD_SIZE)}
      stroke={i % 2 === 0 ? "#6b7280" : "#9ca3af"}
      stroke-width={i % 2 === 0 ? "1.5" : "0.5"}
      opacity="0.3"
    />
    {#if showsLabel(position)}
      <text
        x={x(position)}
        y={y(0) + 15}
        class="fill-gray-600 dark:fill-gray-400 {labelFontSize}"
        text-anchor="middle"
      >
        {$coordinateSettings.coordinateSystem === "ftc" ||
        $coordinateSettings.coordinateSystem === "ftc-inverted"
          ? "Y"
          : "X"}
        {Number(
          toDisplay({ x: position, y: 0 }, $coordinateSettings)[
            $coordinateSettings.coordinateSystem === "ftc" ||
            $coordinateSettings.coordinateSystem === "ftc-inverted"
              ? "y"
              : "x"
          ].toFixed(2),
        )}
        {unitLabel($coordinateSettings)}
      </text>
    {/if}
  {/each}

  <!-- Horizontal grid lines -->
  {#each gridPositions as position, i (position)}
    <line
      x1={x(0)}
      y1={y(position)}
      x2={x(FIELD_SIZE)}
      y2={y(position)}
      stroke={i % 2 === 0 ? "#6b7280" : "#9ca3af"}
      stroke-width={i % 2 === 0 ? "1.5" : "0.5"}
      opacity="0.3"
    />
    {#if showsLabel(position)}
      <text
        x={x(0) - 5}
        y={y(position) + 4}
        class="fill-gray-600 dark:fill-gray-400 {labelFontSize}"
        text-anchor="end"
      >
        {$coordinateSettings.coordinateSystem === "ftc" ||
        $coordinateSettings.coordinateSystem === "ftc-inverted"
          ? "X"
          : "Y"}
        {Number(
          toDisplay({ x: 0, y: position }, $coordinateSettings)[
            $coordinateSettings.coordinateSystem === "ftc" ||
            $coordinateSettings.coordinateSystem === "ftc-inverted"
              ? "x"
              : "y"
          ].toFixed(2),
        )}
        {unitLabel($coordinateSettings)}
      </text>
    {/if}
  {/each}
</svg>
