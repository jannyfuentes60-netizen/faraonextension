
/*
 * FARAÓN MUSIC PROVIDER
 * DJ FARAÓN - VERSION 1.0.0
 * Catálogo: Internet Archive
 */

var cache = {};

function initialize(config) {
    log.info("Faraón Music Provider iniciado");
    return true;
}

function cleanup() {
    cache = {};
}

function permittedLicense(url) {
    var license = String(url || "").toLowerCase();

    return license.includes("creativecommons.org/licenses/") ||
           license.includes("creativecommons.org/publicdomain/");
}

function getMetadata(id) {
    var response = http.get(
        "https://archive.org/metadata/" + encodeURIComponent(id)
    );

    if (!response || !response.ok) return null;

    try {
        return JSON.parse(response.body);
    } catch (e) {
        return null;
    }
}

function searchTracks(query, limit) {
    var q = "mediatype:audio AND (title:(" +
        query + ") OR creator:(" + query + "))";

    var url = "https://archive.org/advancedsearch.php?q=" +
        encodeURIComponent(q) +
        "&fl[]=identifier&fl[]=title&fl[]=creator" +
        "&rows=" + Math.min(Number(limit) || 20, 40) +
        "&output=json";

    var response = http.get(url);

    if (!response || !response.ok) return [];

    var data;

    try {
        data = JSON.parse(response.body);
    } catch (e) {
        return [];
    }

    var docs = data.response.docs || [];
    var results = [];

    docs.forEach(function(doc) {
        var metadata = getMetadata(doc.identifier);

        if (!metadata || !metadata.metadata) return;

        var m = metadata.metadata;
        var license = m.licenseurl || m.license_url;

        if (!permittedLicense(license)) return;

        var track = {
            id: doc.identifier,
            name: m.title || doc.title || "Sin título",
            artists: m.creator || doc.creator || "Desconocido",
            album_name: m.album || "Internet Archive",
            images: "https://archive.org/services/img/" +
                encodeURIComponent(doc.identifier),
            source_url: "https://archive.org/details/" +
                encodeURIComponent(doc.identifier),
            license_url: license
        };

        cache[track.id] = track;
        results.push(track);
    });

    return results;
}

function getTrack(id) {
    if (cache[id]) return cache[id];

    var metadata = getMetadata(id);

    if (!metadata || !metadata.metadata) return null;

    var m = metadata.metadata;
    var license = m.licenseurl || m.license_url;

    if (!permittedLicense(license)) return null;

    return {
        id: id,
        name: m.title || "Sin título",
        artists: m.creator || "Desconocido",
        images: "https://archive.org/services/img/" +
            encodeURIComponent(id),
        license_url: license
    };
}

function getDownloadUrl(id, quality) {
    var metadata = getMetadata(id);

    if (!metadata || !metadata.metadata) {
        return {
            success: false,
            error: "No se encontró la canción."
        };
    }

    var license = metadata.metadata.licenseurl ||
                  metadata.metadata.license_url;

    if (!permittedLicense(license)) {
        return {
            success: false,
            error: "Licencia no compatible."
        };
    }

    var files = metadata.files || [];

    var audio = files.filter(function(file) {
        return /\.(mp3|flac|ogg|wav|m4a|aiff)$/i.test(file.name || "");
    });

    if (quality === "MP3") {
        audio = audio.filter(function(file) {
            return /\.mp3$/i.test(file.name);
        });
    }

    if (!audio.length) {
        return {
            success: false,
            error: "No hay audio disponible."
        };
    }

    var file = audio[0];

    return {
        success: true,
        url: "https://archive.org/download/" +
            encodeURIComponent(id) + "/" +
            encodeURIComponent(file.name),
        filename: file.name,
        format: file.name.split(".").pop(),
        license_url: license
    };
}

registerExtension({
    initialize: initialize,
    cleanup: cleanup,
    searchTracks: searchTracks,
    getTrack: getTrack,
    getDownloadUrl: getDownloadUrl
});

log.info("FARAÓN MUSIC PROVIDER cargado correctamente");
